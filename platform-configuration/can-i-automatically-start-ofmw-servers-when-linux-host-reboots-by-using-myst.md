

Yes, you can start servers automatically by creating the Systemd scripts from MyST by following the steps below

# Prerequisites

- Ensure `oracle` user has access to create `/u01/app/oracle/bin` directory
- Create plain text files in myst [custom action](https://help.mystsoftware.com/platform-configuration/configure-myst-custom-action#creating-custom-actions-in-myst-studio)
   - (name: `ofmw-systemd-control`) (location: `systemd/ofmw-systemd-control.py`)
   
   - paste the code in the **content** section
   
   - ```
     #!/usr/bin/python
     import sys
     from java.io import FileInputStream
     
     # hardcoded
     command = 'start'
     
     def control_ofmw():
         # Setup properties
         print 'ofmw-system-d-control: Initialising properties'
         properties = '/u01/app/oracle/bin/ofmw.properties'
         propInputStream = FileInputStream(properties)
         cfg = Properties()
         cfg.load(propInputStream)
     
         servers = cfg.get('servers').split(',')
         for server in servers:
             domainName = cfg.get('domainName')
             nmHostname = cfg.get('nmHostname')
             serverType = cfg.get('serverType')
             domainHome = cfg.get('domainHome')
             nmPort = int(cfg.get('nmPort'))
             nmUserFile = domainHome + '/nodemanager/userConfigFile'
             nmKeyFile = domainHome + '/nodemanager/userKeyFile'
     
             print 'ofmw-system-d-control: Connecting to nodemanager'
             try:
                 nmConnect(userConfigFile=nmUserFile,userKeyFile=nmKeyFile,host=nmHostname,port=nmPort,domainName=domainName,domainDir=domainHome,verbose='true')
             except Exception, e:
                 dumpStack()
                 print str(e)
     
             # Control OFMW via nodemanager
             if command == 'start':
                 try:
                     if serverType == 'ohs':
                         nmStart(server, serverType='OHS')
                     else:
                         nmStart(server)
                 except WLSTException, e:
                     if 'already running or in the process of starting/restarting' in str(e):
                         print 'ofmw-system-d-control: Process starting or already running: ' + server
                         pass
                     else:
                         print 'ofmw-system-d-control: Error running nmStart() for: ' + server
                         print 'ofmw-system-d-control: ' + str(e)
                         dumpStack()
             elif command == 'stop':
                 print 'ofmw-system-d-control: Stopping OFMW server: ' + server
                 if serverType == 'ohs':
                     nmkill(server, serverType='OHS')
                 else:
                     nmkill(server)
             nmDisconnect()
     
     control_ofmw()
     ```
   
     
   
   - (name: `ofmw-systemd-control-wrapper`) (location: `systemd/ofmw-systemd-control-wrapper.sh`)
   
      - paste the code in the **content** section
   
      - ```
        #!/usr/bin/python
        import sys
        from java.io import FileInputStream
        
        # hardcoded
        command = 'start'
        
        def control_ofmw():
            # Setup properties
            print 'ofmw-system-d-control: Initialising properties'
            properties = '/u01/app/oracle/bin/ofmw.properties'
            propInputStream = FileInputStream(properties)
            cfg = Properties()
            cfg.load(propInputStream)
        
            servers = cfg.get('servers').split(',')
            for server in servers:
                domainName = cfg.get('domainName')
                nmHostname = cfg.get('nmHostname')
                serverType = cfg.get('serverType')
                domainHome = cfg.get('domainHome')
                nmPort = int(cfg.get('nmPort'))
                nmUserFile = domainHome + '/nodemanager/userConfigFile'
                nmKeyFile = domainHome + '/nodemanager/userKeyFile'
        
                print 'ofmw-system-d-control: Connecting to nodemanager'
                try:
                    nmConnect(userConfigFile=nmUserFile,userKeyFile=nmKeyFile,host=nmHostname,port=nmPort,domainName=domainName,domainDir=domainHome,verbose='true')
                except Exception, e:
                    dumpStack()
                    print str(e)
        
                # Control OFMW via nodemanager
                if command == 'start':
                    try:
                        if serverType == 'ohs':
                            nmStart(server, serverType='OHS')
                        else:
                            nmStart(server)
                    except WLSTException, e:
                        if 'already running or in the process of starting/restarting' in str(e):
                            print 'ofmw-system-d-control: Process starting or already running: ' + server
                            pass
                        else:
                            print 'ofmw-system-d-control: Error running nmStart() for: ' + server
                            print 'ofmw-system-d-control: ' + str(e)
                            dumpStack()
                elif command == 'stop':
                    print 'ofmw-system-d-control: Stopping OFMW server: ' + server
                    if serverType == 'ohs':
                        nmkill(server, serverType='OHS')
                    else:
                        nmkill(server)
                nmDisconnect()
        
        control_ofmw()
        ```


- Create a `WLST Action` in the myst custom action`ofmw-systemd-create-ofmw-properties.py` and paste the below code in the **content** section

   ```
   import os
   from com.rubiconred.myst.util import SSHUtil
   
   propertiesDir = '/u01/app/oracle/bin'
   propertiesFile = propertiesDir + '/ofmw.properties'
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
           LOG.info("**************************")
           LOG.info("Starting on node: " + node)
           LOG.info("**************************")
           LOG.debug("Creating ofmw.properties for: " + node)
   
           domainName = cfg.getProperty('core.domain.name')
           productHome = cfg.getProperty('core.fmw.home')
           nmHostname =  cfg.getProperty('core.domain.machine[' + node + '].node-manager.listen-address')
           nmPort = cfg.getProperty('core.domain.machine[' + node + '].node-manager.listen-port')
           nmUser = cfg.getProperty('core.domain.security-configuration.node-manager-username')
           nmPass = cfg.getProperty('core.domain.security-configuration.node-manager-password')
           nmJavaOptions = cfg.getProperty('core.domain.machine[' + node + '].node-manager.custom-java-options')
   
           #serverType
           productId = cfg.getProperty('core.node[' + node + '].product-id')
           LOG.debug('\tDEBUG: productId: ' + productId)
           if productId == 'wls-admin':
               serverType = 'as'
               domainHome = cfg.getProperty('core.fmw.domain-aserver-home')
               nmUserFile = domainHome + '/nodemanager/userConfigFile'
               nmKeyFile = domainHome + '/nodemanager/userKeyFile'
   
               # Generate config/key for aserver nm
               generate_wlst_credentials(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome, nmUserFile, nmKeyFile, node, cfg)
           else:
               if 'webtier' in productId:
                   serverType = 'ohs'
               else:
                   serverType = 'ms'
               LOG.debug('\tDEBUG: serverType: ' + serverType)
               domainHome = cfg.getProperty('core.fmw.domain-mserver-home')
               nmUserFile = domainHome + '/nodemanager/userConfigFile'
               nmKeyFile = domainHome + '/nodemanager/userKeyFile'
               
               # Generate config/key for mserver nm
               generate_wlst_credentials(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome, nmUserFile, nmKeyFile, node, cfg)
   
           #servers
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
           initialise_files(node, cfg)
   
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
           file.close()
           LOG.info('Copying ' + tempPropertiesFile + ' to ' + node)
           SSHUtil.copyFileToNode(node, tempPropertiesFile, propertiesDir, true);
           
           LOG.info("**************************")
           LOG.info("Finishing on node: " + node)
           LOG.info("**************************")
   
   # Remove properties file and create dir
   def initialise_files(node, cfg):
       LOG.info('Initialising files for ' + node)
       LOG.info('Creating directory: ' + propertiesDir)
       SSHUtil.executeCommandOnNode(node, "mkdir -p " + propertiesDir, 0)
       LOG.info('Removing properties if exist: ' + propertiesFile)
       SSHUtil.executeCommandOnNode(node, '[ -f ' + propertiesFile + ' ] && rm -f ' + propertiesFile + ' || echo "ignore"', 0)
       
       # Copy systemd control scripts from admin node
       LOG.info('Copying files: ' + controlFile + ',' + controlWrapperFile)
       SSHUtil.copyFileToNode(node, cfg.getProperty('myst.workspace') + '/resources/custom/systemd/' + controlFile, propertiesDir, true);
       SSHUtil.copyFileToNode(node, cfg.getProperty('myst.workspace') + '/resources/custom/systemd/' + controlWrapperFile, propertiesDir, true);
       SSHUtil.executeCommandOnNode(node, "chmod u+x " + propertiesDir + '/' + controlWrapperFile, 0)
   
   def generate_wlst_credentials(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome, nmUserFile, nmUserKey, node, cfg):
       LOG.info('nmConnect(' + nmUser + ', **********, ' + nmHostname + ', ' + nmPort + ', ' + domainName + ', ' + domainHome + ')')
       nmConnect(nmUser, nmPass, nmHostname, nmPort, domainName, domainHome)
   
       tempUserConfigFile = cfg.getProperty('myst.workspace') + '/userConfigFile'
       tempUserKeyFile = cfg.getProperty('myst.workspace') + '/userKeyFile'
       userConfigHome = domainHome + '/nodemanager/'
       LOG.info('storeUserConfig(' + tempUserConfigFile + ', ' + tempUserKeyFile + ', nm="true")')
       storeUserConfig(tempUserConfigFile, tempUserKeyFile, nm="true")
       LOG.info("Storing temporary encrypted NodeManager " + node + " credentials: " + tempUserConfigFile)
       SSHUtil.copyFileToNode(node, tempUserConfigFile, userConfigHome, true);
       SSHUtil.copyFileToNode(node, tempUserKeyFile, userConfigHome, true);
       LOG.info("Finished running storeUserConfig(). Disconnecting...\n")
       nmDisconnect()
   
   ```
   
- Run Myst custom action `ofmw-systemd-create-ofmw-properties`   to create
   - `/u01/app/oracle/bin/ofmw.properties`
   - `$ASERVER/nodemanager/userConfigFile`
   - `$ASERVER/nodemanager/userKeyFile`
   - `$MSERVER/nodemanager/userConfigFile`
   - `$MSERVER/nodemanager/userKeyFile`

# Configure RHEL

1. Create systemd unit

   `vi /etc/systemd/system/ofmw.service`

2. Initialise ofmw service

   ``` systemctl enable ofmw systemctl daemon-reload systemctl status ofmw ```

3. Start ofmw service

   `systemctl start ofmw`

4. To view ofmw service's log: `journalctl -u ofmw.service`

# Scripts

1. Myst Custom Action `ofmw-systemd-create-ofmw-properties` creates: * `/u01/app/oracle/bin/ofmw-systemd-control-wrapper.sh` * `/u01/app/oracle/bin/ofmw-systemd-control.py` * `/u01/app/oracle/bin/ofmw.properties`
2. `ofmw.service` calls `ofmw-systemd-control-wrapper.sh`
3. `ofmw-systemd-control-wrapper.sh` calls `ofmw-systemd-control.py`
4. `ofmw-systemd-control.py` starts via Oracle's WLST:
5. NodeManager
6. AdminServer/ManagedServer/HTTPServer

## What is ofmw.properties?

This file is generated from the Myst Custom Action `ofmw-systemd-create-ofmw-properties`.

The contents of the file are for the shell and jython scripts to determine how to start the NodeManager and its associated server(s).

   ```
domainName=systemd_domain
productHome=/u01/app/oracle/product/fmw12212
servers=osb_server1,wsm_server1
serverType=ms
domainHome=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain
nmHostname=systemd.mystsoftware.internal
nmPort=5556
nmUserFile=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/nodemanager/userConfigFile
nmKeyFile=/u01/app/oracle/admin/sysd4_domain/mserver/sysd4_domain/nodemanager/userKeyFile
nmJavaOptions=-DLogFile\=/u01/app/oracle/logs/nodemanager/nodemanager-ms1.log

   ```

