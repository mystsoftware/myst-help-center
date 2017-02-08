## {{page.title }} 

Yes.

MyST supports the Automated Deployment of many artifact types:
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

If you wish to deploy an artifact type that is not in the list, you can support this through the extensibility of MyST.

### About Extending MyST

MyST provides an intuitive data model to define your OFMW components. In some scenarios, you might wish to extend MyST in order to achieve specific configuration and/or deployment needs.

MyST provides an extensible framework through declarative WLST or via custom actions to be executed at provision or deploy-time.

Custom actions can be written in Python, Ant, Java or even Puppet manifests.

### Choosing the approach for extending MyST
 
MyST should be extended only when strictly necessary. You should use the out of the box MyST model as much as possible so that your solution is upgrade safe.

If you believe that you cannot achieve an outcome by using the MyST model, please contact Rubicon Red support for advice. The following order of preference should be taken when extending MyST

1. Use the out-of-the-box MyST model
2. Use the Declarative WLST (also know as `myst-extension` in the MyST model)
3. Create a custom action and type

### Create custom types

Custom deployment types can be supported in the MyST CLI through the following property name standard

core.deployment[ID].




Below is an example of how you would go about extending MyST to support deployment of Oracle API Gateway feds and environment settings.

1. Creating out MyST

