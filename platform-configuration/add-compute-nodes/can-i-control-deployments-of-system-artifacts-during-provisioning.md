 Yes, now you can control the deployments of system-artifacts in a phased manner to run pre/post any action.

#  Prerequisites

### System-Artifacts

System-Artifacts are the artifacts which have the dependency on the provisioning life-cycle and they need to be deployed during the provisioning of the environment.

_**Note:-**System-artifact are completely different from the artifacts as they are dependent on the provisioning life-cycle. System artifacts are not controlled from Release Management unlike the usual artifacts and are defined from the Global Variable._

  
_**Example:-**_MQ library is an example of the system-artifact as it needs to be deployed before the action configure-jca to make sure the configuration is appropriate and hence it needs to be deployed during the provisioning of the environment.

# Problem 

How to control the deployment phase of system-artifacts during the provisioning of an environment.

Phase deployment feature is introduced in the** MyST 5.8.2** version and customers using the earlier versions can achieve provisioning-time deployments by defining a Global Variable property **action.create-resource.pre=download-deploy-artifacts,deploy** where **create-resource** is the name of the action.![](/assets/earlier-versions.png)

#### Drawback with earlier approach

The user did not have the control over the deployments of system-artifacts as MyST deployed all the artifacts at a go which caused a few issues.Now, with the MyST versions 5.8.2, each system-artifact can be deployed at a desired level.

_**Example:- **_A user can deploy shared library system-artifact as a pre create-resource action and also deploy a wsm policy as a post copy-domain action.

# Solution

Deployments of system-artifacts can be achieved in two ways, either from the MyST Studio or from MyST CLI. The property**`core.deployment[ID].param[phase]`**is used to define the stage of deployment of the system-artifact where ID denotes the unique identifier for the artifact instance.  


#### MyST Studio

* Login to MyST Studio, navigate to **Platform Blueprint/Platform Model -&gt; Edit Configuration -&gt;Global Variable**.
* Add the** core.deployment** properties of the system-artifact as shown in the below screenshot.![](/assets/phase-parameter.png)

* The property highlighted above **`core.deployment[ID].param[phase]`**defines the phase where the system-artifact has to be deployed, you can either set it to pre/post to any action in the provisioning life-cycle.
* For instance if you want to schedule the deployment before the action create-resource then you can set it as shown in the above screenshot.
* Go to **Platform Model -&gt; Action -&gt; Provision**
* This will deploy the system-artifact during the provisioning of the environment. 

####  MyST CLI

* Define the core.properties for the deployment under the **mystWorkspace/conf/fc.properties** as shown below.
  * `core.deployment[ID].artifact.repository.artifactId=wspolicy`
    `core.deployment[ID].artifact.repository.groupId=com.rubiconred`
    `core.deployment[ID].artifact.repository.type=jar`
    `core.deployment[ID].artifact.repository.version=1.0-3`
    `core.deployment[ID].param[extract-files]=policy`
    `core.deployment[ID].param[phase]=create-resource-post`
    `core.deployment[ID].type=wspolicy`

* The property highlighted above **`core.deployment[ID].param[phase]`** defines the phase where the system-artifact has to be deployed, you can either set it to pre/post to any action in the provisioning life-cycle.
* For instance if you want to schedule the deployment after the action create-resource then you can set the parameter as shown in the above properties.
* Run the provision action from the MyST CLI.
* This will deploy the system-artifact during the provisioning of the environment.



