## {{page.title }} 

Yes.

If you wish to deploy an artifact type that is not in the list of supported MyST Artifact Types, you can do this through the MyST Extensibility Framework.

### About Extending MyST

MyST provides an intuitive data model to define your OFMW components. In some scenarios, you might wish to extend MyST in order to achieve specific configuration and/or deployment needs.

MyST provides an extensible framework through declarative WLST or via custom actions to be executed at provision or deploy-time.

Custom actions can be written in Python, Jython, WLST, Ant, Java or even Puppet manifests.

### Choosing the approach for Extending MyST
 
MyST should be extended only when strictly necessary. You should use the out of the box MyST model as much as possible so that your solution is upgrade safe.

If you believe that you cannot achieve an outcome by using the MyST model, please contact Rubicon Red support for advice. The following order of preference should be taken when deciding how to model configuration and deployment within MyST.

1. Use the out-of-the-box definitions. MyST supports the Automated Deployment of many artifact types:
  * Oracle ADF     
  * Application Configuration
  * Oracle B2B
  * Oracle BAM
  * Java EAR
  * Java JAR
  * Java Library (Classpath)
  * Java WAR
  * Oracle MDS
  * Oracle MFT
  * OSB Configuration
  * OSB XPath
  * Oracle WSM Policy
  * Oracle SOA Composites
  * Oracle SOA Extension
  * PL/SQL
2. If the desired type is not supported, use MyST's Declarative Extension Model (also know as `myst-extension`).
3. If the type can not be supported through `myst-extension` consider creating a custom action and type or raise it as a feature request with MyST Support.

### Creating Custom Types

Custom deployment types can be supported in the MyST CLI through the following property name standard where 
 * `ID` is a unique identifier for the artifact instance, 
 * `TYPE-ID` is a unique identifier for the artifact type and
 * `PARAM-ID` is a unique identifier for any artifact parameters

```
core.deployment[ID].artifact.repository.artifactId
core.deployment[ID].artifact.repository.groupId
core.deployment[ID].artifact.repository.version
core.deployment[ID].present
core.deployment[ID].type=TYPE-ID
core.deployment[ID].param[PARAM-ID]
```

For example, if we wanted to create an artifact type for an Oracle API Gateway Federation we may use the following:

```
core.deployment[OAG_CONFIG
].artifact.repository.artifactId=OAG_CONFIG
core.deployment[OAG_CONFIG
].artifact.repository.groupId=com.acme
core.deployment[OAG_CONFIG
].artifact.repository.version=1.0.0-1
core.deployment[OAG_CONFIG
].present=true
core.deployment[OAG_CONFIG
].type=oag-fed
core.deployment[OAG_CONFIG
].param[target-groups]=default_group
```

As with all MyST CLI resources, you can alternatively use XML instead of Name/Value Properties if you want:

```
<deployment id="OAG_CONFIG">
  <artifact>  
    <repository>
      <artifactId>OAG_CONFIG</artifactId>  
      <groupId>com.acme</groupId>  
      <version>1.0.0-1</version>  
      <type>jar</type>  
    </repository>  
  </artifact>  
  <present>true</present>
  <type>sql</type>
  <param-list>
    <param id="target-groups">default_group</param>
  </param-list>
</deployment>
```  

If you want to define your artifact instance properties within an existing Platform Blueprint or Model in **MyST Studio** you should use the name/value property notation define it under the **Global Variables**.

### Creating Custom Actions

MyST supports the injection of Custom Actions within various parts of the Provisioning and Deployment workflow. 

When using the CLI, custom actions should be defined and version-controlled under a specific folder within the MyST workspace depending on the language they are written in:

| Language | Location | File Extension
| ------------- | -------------- | -------------- |
| Python / Jython | ext/targets/jython | .py |
| WLST Script | ext/targets/wlst | .py |
| Java Archive | ext/lib/thirdparty | .jar |
| Puppet Manifest | ext/lib/puppet | .pp |

At runtime, MyST CLI will auto-discover these custom actions. To inject them before or after specific out-of-the-box MyST actions you can use the following property notation:

```
action.<ACTION NAME>.pre|post
```

For example, to execute an action called `deploy-oag` after all of the other deployments are complete you could set the following:

```
action.deploy.post=deploy-oag
```

#### Python / Jython / WLST

In terms of the scripting language and how it interacts with MyST, Python, Jython and WLST are all quite similar. The differences are as follows:
 * Jython code is written in Python but it runs in the Java Virtual Machine so has native access to Java Objects
 * WLST code is written in Jython so it has access to the additional WebLogic functions. WLST actions have a dependency on WebLogic being installed so if you do not need access to the WebLogic functions you should opt for a Python or Jython action.
 
The action name will be the same name as the file, without the file extension. For example, if the file is called `configure-mediator.py`, then the action name will be `configure-mediator`.

At runtime, MyST passes all of the properties via the `conf` context. In case where the values to be set in the action are either different per environment or could be changed often, it's recommended to externalise them into your model.

##### Example 1: Configure Mediator MBeans via WLST

`ext/targets/wlst/configure-mediator.py`
```
from java.util import HashMap 
   
def myst(conf): 
  if 'admin.host' in conf and 'admin.port' in conf and 'core.fmw.admin.username' in conf and 'core.fmw.admin.password' in conf: 
  LOG.info('Connecting to Admin Server') 
  admin_server_url=conf['admin.host'] + ':' + conf['admin.port'] 
  connect(conf['core.fmw.admin.username'],conf['core.fmw.admin.password'],admin_server_url) 
  domainRuntime() 
  if 'servers' in conf: 
    servers=conf['servers'] 
    serversserversList=servers.split(',') 
    for server in serversList: 
      serverName = conf["core.domain.server["+server+"].name"] 
        mediator= ObjectName('oracle.as.soainfra.config:Location='+serverName+',name=mediator,type=MediatorConfig,Application=soa-infra') 
        print 'DeferredWorkerThreadCount:' 
        print mbs.getAttribute(mediator, 'DeferredWorkerThreadCount') 
        mbs.setAttribute(mediator, Attribute('DeferredWorkerThreadCount',2)) 
   
  disconnect() 
```

##### Example 2: Deploy Oracle API Gateway Federations

`ext/targets/wlst/deploy-oag.py`
```
execfile(System.getProperty("myst.home") + '/lib/targets/common/utils.py')

def myst(cfg):
    if not 'oag.deployment.federations' in cfg:
        return

    scriptName = 'deploy-oag-fed-archive.py'

    deployFedsProp = cfg['oag.deployment.federations']
    deployFedsList = deployFedsProp.split(',')

    cfg['oag.remote.tmpdir'] = '/tmp/myst'
    remoteTmpDir = cfg['oag.remote.tmpdir']

    adminNode = cfg['oag.admin.node-id']
    adminHost = cfg['core.node[' + adminNode + '].host']
    cfg['remote.server.host'] = adminHost
    cfg['remote.server.username'] = cfg['core.node[' + adminNode + '].authentication.credentials.username']
    cfg['remote.server.password'] = cfg['core.node[' + adminNode + '].authentication.credentials.password']
    cfg['destination.directory'] = cfg['oag.remote.tmpdir']
    scriptLocation = System.getProperty('myst.workspace') + '/resources/oag'
    cfg['library.file'] = scriptLocation + '/' + scriptName

    if not 'noop' in cfg:
        call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-copy-unix')

    remoteCommand = 'mkdir -p ' + remoteTmpDir
    cfg['remote.execute.command'] = remoteCommand
    if not 'noop' in cfg:
        call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-execute')

    for deployFed in deployFedsList:
      
        sourceLocation = cfg['oag.deployment.federation.' + deployFed + '.source.location']
        sourceFile = cfg['oag.deployment.federation.' + deployFed + '.source.file']
        targetGroups = cfg['oag.deployment.federation.' + deployFed + '.target.groups']

        cfg['destination.directory'] = remoteTmpDir
        cfg['library.file'] = sourceLocation + '/' + sourceFile

        print 'Copy file ' + cfg['destination.directory'] + '/' + cfg['library.file']

        if not 'noop' in cfg:
            call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-copy-unix')

        targetGroupsList = targetGroups.split(',')
        for targetGroup in targetGroupsList:

            jythonFile = cfg['core.product[oag].home'] + '/apigateway/posix/bin/jython'

            remoteCommand = 'cd ' + remoteTmpDir  + '; ' + jythonFile + ' ' + scriptName + ' ' + adminHost + ' ' + cfg['core.product[oag].param[node-manager-port]'] + ' "' + cfg['core.product[oag].param[password]'] + '" ' + remoteTmpDir + '/' + sourceFile + ' ' + targetGroup
            cfg['remote.execute.command'] = remoteCommand

            if not 'noop' in cfg:
                call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-execute')
			
            remoteCommand = 'rm -f ' + remoteTmpDir + '/' + sourceFile
            cfg['remote.execute.command'] = remoteCommand
            if not 'noop' in cfg:
                call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-execute')

    remoteCommand = 'rm -f ' + remoteTmpDir + '/' + scriptName
    cfg['remote.execute.command'] = remoteCommand
    if not 'noop' in cfg:
        call_ant(cfg, '/lib/targets/ant/deploy-libs.xml','remote-execute')
```

#### Java

The JAR files for the MyST extension and all the dependent JARs must be under `<MYST_WORKSPACE>/lib/thirdparty`.

If some JARS must be loaded before the MyST JARs, then these files must be placed under `<MYST_WORKSPACE>/lib/thirdparty/pre`.
 

