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

When using the CLI, custom actions should be defined under a specific folder within the MyST workspace depending on the language they are written in:

|| Language || Location ||
| Python/Jython | ext/targets |

#### Python / Jython

.....

#### WLST

The python script must be created under <MYST_WORKSPACE>/ext/targets
Make sure that the custom action scripts are kept under version control
The action name will be the same name as the python file, without the file extension. For example, if the file is called configure-mediator.py, then the action name will be configure-mediator
In the following example the MBean attribute will be set for all the WLS servers. 
Note that any MyST properties can be accessed from the conf context
In case the values to be set in the MBeans are either different per environment or could be changed often, it's recommended to externalise them in your environment property files.


