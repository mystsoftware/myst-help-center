

Yes, you can start and stop servers gracefully and automatically by creating the Systemd scripts from MyST by following the steps below.

# Prerequisites

- `oracle` user has access to create directories:

   - `/u01/app/oracle/bin` 
   - `/u01/app/oracle/admin/shared/systemd`



# Configure Myst

Create plain text files in the Myst [custom action](https://help.mystsoftware.com/platform-configuration/configure-myst-custom-action#creating-custom-actions-in-myst-studio).

![](img\systemd_custom_actions.png)

### OFMW Systemd Control

   - **Name** `ofmw-systemd-control`

   - **Type** Plain Text

   - **Location** `systemd/ofmw-systemd-control.py`

```python
#!/usr/bin/python
import sys
from java.io import FileInputStream

# start or stop
command = sys.argv[1]

def control_ofmw():
    # Setup systemd properties
    print 'ofmw-systemd-control: Initialising systemd properties'
    properties_systemd = '/u01/app/oracle/bin/systemd.properties'
    propInputStream = FileInputStream(properties_systemd)
    cfg_systemd = Properties()
    cfg_systemd.load(propInputStream)
    nodeDNS = cfg_systemd.getProperty('dns')
    
    # Load properties
    print 'ofmw-systemd-control: Initialising ofmw properties'
    properties = '/u01/app/oracle/admin/shared/systemd/' + nodeDNS + '/ofmw.properties'
    propInputStream = FileInputStream(properties)
    cfg = Properties()
    cfg.load(propInputStream)

    # Retrieve properties
    domainName = cfg.get('domainName')
    nmHostname = cfg.get('nmHostname')
    serverType = cfg.get('serverType')
    domainHome = cfg.get('domainHome')
    aserverDomainHome = cfg.get('aserverDomainHome')
    nmPort     = int(cfg.get('nmPort'))
    nmUserFile = cfg.get('nmUserFile')
    nmKeyFile  = cfg.get('nmKeyFile')
    
    # Retrieve AdminServer connection properties
    url        = cfg.get('adminUrl')
    asUserFile = aserverDomainHome + '/admin_credentials/as_userConfigFile'
    asKeyFile  = aserverDomainHome + '/admin_credentials/as_userKeyFile'

    #####
    # Begin start/stop by iterating Myst CLI server list
    #####
    servers = cfg.get('servers').split(',')
    for server in servers:
        # START
        if command == 'start':
            try:
                nmConnect(userConfigFile=nmUserFile,userKeyFile=nmKeyFile,host=nmHostname,port=nmPort,domainName=domainName,domainDir=domainHome,verbose='true')
                if serverType == 'ohs':
                    nmStart(server, serverType='OHS')
                else:
                    nmStart(server)
            except WLSTException, e:
                if 'already running or in the process of starting/restarting' in str(e):
                    print 'ofmw-systemd-control: Process starting or already running: ' + server
                    pass
                else:
                    print 'ofmw-systemd-control: Error running nmStart() for: ' + server
                    print 'ofmw-systemd-control: ' + str(e)
                    dumpStack()
                nmDisconnect()
        # STOP
        elif command == 'stop':
            print 'ofmw-systemd-control: Stopping OFMW server: ' + server
            if validate_adminserver_running(servers, asUserFile, asKeyFile, url):
                stop_via_as(cfg, server)
            else:
                print 'ofmw-systemd-control: Shutdown forcefully via nodemanager'
                nmConnect(userConfigFile=nmUserFile,userKeyFile=nmKeyFile,host=nmHostname,port=nmPort,domainName=domainName,domainDir=domainHome,verbose='true')
                if serverType == 'ohs':
                    nmKill(server, serverType='OHS')
                else:
                    nmKill(server)
                nmDisconnect()

# Validate AdminServer is running
def validate_adminserver_running(servers, asUserFile, asKeyFile, url):
    try:
        if servers is 'admin.server':
            return False
        print 'ofmw-systemd-control: Attempting to connect to AdminServer'
        connect(userConfigFile=asUserFile,userKeyFile=asKeyFile,url=url)
        return True
    except:
        print 'ofmw-systemd-control: Unable to connect AdminServer for shutdown() gracefully.'
        print 'ofmw-systemd-control: Proceeding with NodeManager nmKill() instead.'
        return False

# Graceful shutdown via AdminServer
def stop_via_as(cfg, server):
    timeout = cfg.get('timeout')
    print "ofmw-systemd-control: shutdown gracefully via adminserver"
    # shutdown MS01, ignoreSessions=false, timeout=300sec, force=false, block=false
    print "ofmw-systemd-control: shutdown(name=" + server + ",ignoreSessions='true',timeOut=" + timeout + ",force='false',block='true')"
    try:
        shutdown(name=server,ignoreSessions='true',timeOut=int(timeout),force='false',block='true')
    except Exception, e:
        print "ofmw-systemd-control: Error shutting down " + server + ". It may already be shutdown."
        print "ofmw-systemd-control: SKIPPING..."

# Main
control_ofmw()

```




### OFMW Systemd Control Wrapper

   - **name** `ofmw-systemd-control-wrapper`

   - **Type** `Plain Text`

   - **Location** `systemd/ofmw-systemd-control-wrapper.sh`

```shell
#!/bin/bash

# $SYSTEMD_PROPERTIES is generated by AWS autoscaling lifecycle scripts with expected content:
# dns=<DNS_name_of_ec2_instance_used_by_OFMW_and_Myst>
# dns=oal-soa1.sit4.com
SYSTEMD_PROPERTIES="/u01/app/oracle/bin/systemd.properties"
if [ -f "$SYSTEMD_PROPERTIES" ]; then
    source $SYSTEMD_PROPERTIES
else
    echo "ERROR: Could not locate $SYSTEMD_PROPERTIES"
    exit 1
fi

# $dns environment variable is sourced from $SYSTEMD_PROPERTIES
SCRIPT_HOME=/u01/app/oracle/admin/shared/systemd
OFMW_PROPERTIES=$SCRIPT_HOME/$dns/ofmw.properties
productHome=`grep "productHome" $OFMW_PROPERTIES | cut -d'=' -f2`
domainHome=`grep "domainHome" $OFMW_PROPERTIES | cut -d'=' -f2`
nmPort=`grep "nmPort" $OFMW_PROPERTIES | cut -d'=' -f2`
nmJavaOptions=`grep "nmJavaOptions" $OFMW_PROPERTIES | cut -c 15-`

# Start NodeManager
if [ $1 = "start" ]; then
    echo -e "\nofmw-systemd-control-wrapper: Starting NodeManager"
    echo -e "ofmw-systemd-control-wrapper: Java options (if applicable): $nmJavaOptions"
    export JAVA_OPTIONS=$nmJavaOptions
    nohup $domainHome/bin/startNodeManager.sh > /dev/null 2>&1 &
fi

# check nodemanager port is running before continuing
NM_STATUS=`netstat -anp | grep ":$nmPort " | grep LISTEN`
RESULT=$?
i=0
while [ $RESULT -ne 0 ]; do
    # break if over 5 minutes
    ((i++))
    if [[ "$i" == 60 ]]; then
        break
    fi
    
    sleep 5
    NM_STATUS=`netstat -anp | grep ":$nmPort " | grep LISTEN`
    RESULT=$?
done

# Start OFMW
echo -e "\nofmw-systemd-control-wrapper: $1 OFMW servers"
source $productHome/wlserver/server/bin/setWLSEnv.sh
java weblogic.WLST $SCRIPT_HOME/ofmw-systemd-control.py $1

```



### OFMW Properties Creation

- **Name** `ofmw-systemd-create-ofmw-properties.py`

- Type `wlst`

```python
import os
from com.rubiconred.myst.util import SSHUtil

systemdPropertiesBaseDir = '/u01/app/oracle/bin'
systemdProperties = systemdPropertiesBaseDir + '/systemd.properties'
ofmwPropertiesBaseDir = '/u01/app/oracle/admin/shared/systemd'
ofmwPropertiesFilename = 'ofmw.properties'
controlFile = 'ofmw-systemd-control.py'
controlWrapperFile = 'ofmw-systemd-control-wrapper.sh'

def myst(cfg):
    """Creates NM encrypted config/key and initialises the systemd scripts and properties"""

    nodes = cfg.getProperty('nodes').split(',')
    nodeDict = {}
    for node in nodes:
        nodeDict[node] = []

    # add server to nodes dictionary
    servers = cfg.getProperty('servers')
    if servers is not None:
        for server in servers.split(','):
            machineId = cfg.getProperty('core.domain.server[' + server + '].machine-id')
            LOG.debug('\tDEBUG: adding ' + server + ' to ' + machineId)
            nodeDict[machineId].append([server])

    # add ohs to nodes dictionary
    ohsServers = cfg.getProperty('system-components')
    if ohsServers is not None:
        for ohsServer in ohsServers.split(','):
            machineId = cfg.getProperty('core.domain.system-component[' + ohsServer + '].machine-id')
            LOG.debug('\tDEBUG: adding ' + ohsServer + ' to ' + machineId)
            nodeDict[machineId].append([ohsServer])

    # create properties file
    nodes = cfg.getProperty('nodes').split(',')
    for node in nodes:
        try:
            LOG.info("**************************")
            LOG.info("Starting on node: " + node)
            LOG.info("**************************")
            LOG.debug("Creating ofmw.properties for: " + node)

            # nodemanager configuration
            domainName = cfg.getProperty('core.domain.name')
            productHome = cfg.getProperty('core.fmw.home')
            nmHostname =  cfg.getProperty('core.domain.machine[' + node + '].node-manager.listen-address')
            nmPort = cfg.getProperty('core.domain.machine[' + node + '].node-manager.listen-port')
            nmUser = cfg.getProperty('core.domain.security-configuration.node-manager-username')
            nmPass = cfg.getProperty('core.domain.security-configuration.node-manager-password')
            nmJavaOptions = cfg.getProperty('core.domain.machine[' + node + '].node-manager.custom-java-options')
            if nmJavaOptions is None:
                nmJavaOptions = ""
            
            # Node Name
            nodeDNS = cfg.getProperty('core.domain.machine[' + node + '].node-manager.listen-address')
            ofmwPropertiesDir = ofmwPropertiesBaseDir + '/' + nodeDNS

            # Create systemd.properties if does not exist (ie. not created by AWS Autoscale Lifecycle)
            LOG.info('Creating ' + node + ':' + systemdProperties)
            SSHUtil.executeCommandOnNode(node, 'mkdir -p ' + systemdPropertiesBaseDir + '; echo "dns=' + nodeDNS + '" > ' + systemdProperties, 0)

            #serverType
            productId = cfg.getProperty('core.node[' + node + '].product-id')
            LOG.debug('\tDEBUG: productId: ' + productId)
            
            # AdminServer - this code only runs *once*
            if productId == 'wls-admin':
                serverType = 'as'
                domainHome = cfg.getProperty('core.fmw.domain-aserver-home')

                # NodeManager credentials
                nmUserFile = domainHome + '/nodemanager/' + nodeDNS + '/userConfigFile'
                nmKeyFile = domainHome + '/nodemanager/' + nodeDNS + '/userKeyFile'

                # Generate AdminServer credentials and placed in mserver shared storage
                # The user/key is used by managed servers to connect to the AdminServer for graceful shutdown
                mserver_home = cfg.getProperty('core.fmw.domain-mserver-home')
                asUserFile = mserver_home + '/admin_credentials/userConfigFile'
                asUserKey  = mserver_home + '/admin_credentials/userKeyFile'    
                generate_wlst_as_credentials(domainName, domainHome, asUserFile, asUserKey, node, cfg)

            # mserver or system components
            else:
                if 'webtier' in productId:
                    serverType = 'ohs'
                else:
                    serverType = 'ms'
                LOG.debug('\tDEBUG: serverType: ' + serverType)
                domainHome = cfg.getProperty('core.fmw.domain-mserver-home')
                nmUserFile = domainHome + '/nodemanager/' + nodeDNS + '/userConfigFile'
                nmKeyFile = domainHome + '/nodemanager/' + nodeDNS + '/userKeyFile'
                
            # Generate config/key for nodemanager per host (excluding adminserver)
            generate_wlst_nm_credentials(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome, nmUserFile, nmKeyFile, node, nodeDNS, cfg)

            # Construct a list of servers on the node
            serverList = map(''.join,nodeDict[node])
            servers = None
            LOG.debug('serverList: ' + str(serverList))
            for serverNames in serverList:
                productId = cfg.getProperty('core.node[' + node + '].product-id')
                LOG.debug('\tDEBUG: serverNames: ' + str(serverNames))
                LOG.debug('\tDEBUG: serverType: ' + serverType)
                if servers is None:
                    if serverType is 'ohs':
                        servers = cfg.getProperty('core.domain.system-component[' + serverNames + '].name')
                    else:
                        servers = cfg.getProperty('core.domain.server[' + serverNames + '].name')
                else:
                    if serverType is 'ohs':
                        servers = servers + ',' + cfg.getProperty('core.domain.system-component[' + serverNames + '].name')
                    else:
                        servers = servers + ',' + cfg.getProperty('core.domain.server[' + serverNames + '].name')

            # create ofmw.properties
            initialise_files(node, ofmwPropertiesDir, nodeDNS, cfg)

            LOG.info('Creating ofmw.properties on: ' + node)
            tempPropertiesFile = cfg.getProperty('myst.workspace') + '/ofmw.properties'
            file = open(tempPropertiesFile, 'w')
            file.write('domainName='    + domainName + '\n')
            file.write('productHome='   + productHome + '\n')
            file.write('servers='       + servers + '\n')
            file.write('serverType='    + serverType + '\n')
            file.write('domainHome='    + domainHome + '\n')
            file.write('nmHostname='    + nmHostname + '\n')
            file.write('nmPort='        + nmPort + '\n')
            file.write('nmUserFile='    + nmUserFile + '\n')
            file.write('nmKeyFile='     + nmKeyFile + '\n')
            file.write('nmJavaOptions=' + nmJavaOptions + '\n')
            file.write('aserverDomainHome=' + cfg.getProperty('core.fmw.domain-aserver-home') + '\n')
            file.write('asUserFile='    + asUserFile + '\n')
            file.write('asUserKey='     + asUserKey + '\n')
            file.write('adminUrl='      + cfg.getProperty('admin.url') + '\n')
            file.write('timeout='       + get_timeout(cfg) + '\n')
            file.close()
            LOG.info('Copying ' + tempPropertiesFile + ' to ' + node + ':' + ofmwPropertiesDir)
            SSHUtil.copyFileToNode(node, tempPropertiesFile, ofmwPropertiesDir, true);

            LOG.info("**************************")
            LOG.info("Finishing on node: " + node)
            LOG.info("**************************")
        except Exception, e:
            if is_autoscale_enabled(cfg):
                LOG.warn('validation.ssh is set to "false" hence ignoring the error as the host may be unavailable.')
                continue
            else:
                raise(e)

def initialise_files(node, ofmwPropertiesDir, nodeDNS, cfg):
    LOG.info('Initialising files for ' + node)
    LOG.info('Creating directory: ' + ofmwPropertiesDir)
    
    # Remove properties file and create dir
    ofmwPropertiesFile = ofmwPropertiesDir + '/' + nodeDNS + '/' + ofmwPropertiesFilename
    SSHUtil.executeCommandOnNode(node, "mkdir -p " + ofmwPropertiesDir, 0)
    LOG.info('Removing properties if exist: ' + ofmwPropertiesFile)
    SSHUtil.executeCommandOnNode(node, '[ -f ' + ofmwPropertiesFile + ' ] && rm -f ' + ofmwPropertiesFile + ' || echo "ignore"', 0)
    
    # Copy systemd control scripts from admin node to node-specific folder
    LOG.info('Copying files: ' + controlFile + ',' + controlWrapperFile)
    SSHUtil.copyFileToNode(node, cfg.getProperty('myst.workspace') + '/resources/custom/systemd/' + controlFile, ofmwPropertiesBaseDir, true);
    SSHUtil.copyFileToNode(node, cfg.getProperty('myst.workspace') + '/resources/custom/systemd/' + controlWrapperFile, ofmwPropertiesBaseDir, true);
    SSHUtil.executeCommandOnNode(node, "chmod u+x " + ofmwPropertiesBaseDir + '/' + controlWrapperFile, 0)

# Create nodemanager user and key file for passwordless starting of managed servers
def generate_wlst_nm_credentials(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome, nmUserFile, nmUserKey, node, nodeDNS, cfg):
    LOG.info('nmConnect(' + nmUser + ', **********, ' + nmHostname + ', ' + nmPort + ', ' + domainName + ', ' + domainHome + ')')
    nmConnect(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome)

    tempUserConfigFile = cfg.getProperty('myst.workspace') + '/userConfigFile'
    tempUserKeyFile = cfg.getProperty('myst.workspace') + '/userKeyFile'
    userConfigHome = domainHome + '/nodemanager/' + nodeDNS
    LOG.info('storeUserConfig(' + tempUserConfigFile + ', ' + tempUserKeyFile + ', nm="true")')
    storeUserConfig(tempUserConfigFile, tempUserKeyFile, nm="true")
    LOG.info("Storing temporary encrypted NodeManager " + node + " credentials: " + tempUserConfigFile)
    
    # Check if the files are generated. If not, the user may need to have '-Dweblogic.management.confirmKeyfileCreation=true' enabled
    if not os.path.isfile(tempUserConfigFile) and not os.path.isfile(tempUserKeyFile):
        LOG.error('Cannot find files: ' + tempUserConfigFile + ' and ' + tempUserKeyFile)
        LOG.error('Add the a global variable in the Myst Platform Blueprint so the keys can generate without user input')
        LOG.error('myst.system.properties=-Dweblogic.management.confirmKeyfileCreation=true')
        raise Error('*****\n*****\nCannot generate key files. See previous log messages for instructions to add "-Dweblogic.management.confirmKeyfileCreation=true"\n*****\n*****')
    
    # Copy user and key file to domain_home/nodemanager/<node_dns_name>/
    SSHUtil.copyFileToNode(node, tempUserConfigFile, userConfigHome, true);
    SSHUtil.copyFileToNode(node, tempUserKeyFile, userConfigHome, true);
    LOG.info("Finished running storeUserConfig(). Disconnecting...\n")
    nmDisconnect()

# Create nodemanager user and key file for passwordless starting of managed servers
def generate_wlst_as_credentials(domainName, domainHome, asUserFile, asUserKey, node, cfg):
    LOG.info('connect(' + cfg.getProperty('core.fmw.admin.username') + ', **********, ' + cfg.getProperty('admin.url'))
    connect(cfg.getProperty('core.fmw.admin.username'), cfg.getProperty('core.fmw.admin.password'), cfg.getProperty('admin.url'))

    tempUserConfigFile = cfg.getProperty('myst.workspace') + '/as_userConfigFile'
    tempUserKeyFile = cfg.getProperty('myst.workspace') + '/as_userKeyFile'
    userConfigHome = domainHome + '/admin_credentials/'
    LOG.info('storeUserConfig(' + tempUserConfigFile + ', ' + tempUserKeyFile + ', nm="false")')
    storeUserConfig(tempUserConfigFile, tempUserKeyFile, nm="false")
    LOG.info("Storing temporary encrypted AdminServer credentials: " + tempUserConfigFile)
    
    # Check if the files are generated. If not, the user may need to have '-Dweblogic.management.confirmKeyfileCreation=true' enabled
    if not os.path.isfile(tempUserConfigFile) and not os.path.isfile(tempUserKeyFile):
        LOG.error('Cannot find files: ' + tempUserConfigFile + ' and ' + tempUserKeyFile)
        LOG.error('Add the a global variable in the Myst Platform Blueprint so the keys can generate without user input')
        LOG.error('myst.system.properties=-Dweblogic.management.confirmKeyfileCreation=true')
        raise Error('*****\n*****\nCannot generate key files. See previous log messages for instructions to add "-Dweblogic.management.confirmKeyfileCreation=true"\n*****\n*****')
    
    # Copy user and key file to domain_home/nodemanager/<node_dns_name>/
    SSHUtil.copyFileToNode(node, tempUserConfigFile, userConfigHome, true);
    SSHUtil.copyFileToNode(node, tempUserKeyFile, userConfigHome, true);
    LOG.info("Finished running storeUserConfig(). Disconnecting...\n")
    disconnect()

# Check if autoscaling is enabled
def is_autoscale_enabled(cfg):
    validationSSH = cfg.getProperty('validation.ssh')
    if validationSSH is not None and validationSSH == 'false':
        return true
    return false

# Get timeout property for graceful shutdown
def get_timeout(cfg):
    timeout = cfg.getProperty('timeout')
    if timeout is None:
        timeout = '600000'
    else:
        # convert seconds to milliseconds
        timeout = timeout + "000"
    return timeout

```

- Run Myst custom action `ofmw-systemd-create-ofmw-properties` to create
   - `/u01/app/oracle/bin/ofmw-systemd-control-wrapper.sh`
   - `/u01/app/oracle/bin/ofmw-systemd-control.py`
   - `/u01/app/oracle/bin/ofmw.properties`
   - `$ASERVER/nodemanager/userConfigFile`
   - `$ASERVER/nodemanager/userKeyFile`
   - `$MSERVER/nodemanager/userConfigFile`
   - `$MSERVER/nodemanager/userKeyFile`
   - `$MSERVER/nodemanager/as_userConfigFile`
   - `$MSERVER/nodemanager/as_userKeyFile`



# Configure RHEL

1. Create systemd unit file

   `vi /etc/systemd/system/ofmw.service`

   ```
   [Unit]
   Description=Oracle Fusion Middleware control for NM, AS, MS, and OHS
   After=syslog.target network.target cloud-final.service
   
   [Service]
   Type=oneshot
   WorkingDirectory=/u01/app/oracle/admin/shared/systemd
   ExecStart=/u01/app/oracle/admin/shared/systemd/ofmw-systemd-control-wrapper.sh "start"
   ExecStop=/u01/app/oracle/admin/shared/systemd/ofmw-systemd-control-wrapper.sh "stop"
   User=oracle
   Group=oracle
   RemainAfterExit=yes
   
   [Install]
   WantedBy=multi-user.target
   ```

2. Initialise ofmw service

   ``` systemctl enable ofmw systemctl daemon-reload systemctl status ofmw ```

3. Start or stop ofmw service

   `systemctl start ofmw`
   `systemctl stop ofmw`

4. To view ofmw service's log

   `journalctl -u ofmw.service`
   
5. Remove `cloud-final.service` from the unit file if you are not using AWS.



# Scripts

Myst Custom Action `ofmw-systemd-create-ofmw-properties` creates: 

1. `/u01/app/oracle/bin/ofmw-systemd-control-wrapper.sh` 
2. `/u01/app/oracle/bin/ofmw-systemd-control.py` 
3. `/u01/app/oracle/bin/ofmw.properties`



Systemd script's workflow:

1. `ofmw.service` -> `ofmw-systemd-control-wrapper.sh`
2. `ofmw-systemd-control-wrapper.sh` -> `ofmw-systemd-control.py`
3. `ofmw-systemd-control.py` -> Start/Stop `NodeManager / AdminServer / Managed Server / HTTP Server`

## What are ofmw.properties and systemd.properties?

### ofmw.properties

This file is generated from the Myst Custom Action `ofmw-systemd-create-ofmw-properties`.

The contents of the file are for the shell and jython scripts to determine how to start the NodeManager and its associated server(s).

```shell
domainName=systemd_domain
productHome=/u01/app/oracle/product/fmw12212
servers=osb_server1,wsm_server1
serverType=ms
domainHome=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain
nmHostname=host_ms1.mystsoftware.internal
nmPort=5556
nmUserFile=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/nodemanager/userConfigFile
nmKeyFile=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/nodemanager/userKeyFile
nmJavaOptions=-DLogFile\=/u01/app/oracle/logs/nodemanager/nodemanager-ms1.log
aserverDomainHome=/u01/app/oracle/admin/sysd4_domain/aserver/sysd4_domain
asUserFile=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/admin_credentials/as_userConfigFile
asUserKey=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/admin_credentials/as_userKeyFile
adminUrl=host_as.mystsoftware.internal:7001
timeout=600
```

### systemd.properties

The **Myst Custom Action** `ofmw-systemd-create-ofmw-properties`  generates this file if it does not exist. Generally this file should be generated by the Linux environment on instance creation.

The contents helps the systemd OFMW startup scripts determine which node the script is running on by using the **nodemanager's listen address**.



Example of the file `/u01/app/oracle/bin/systemd.properties`

```shell
node=host_ms1.mystsoftware.internal
```

