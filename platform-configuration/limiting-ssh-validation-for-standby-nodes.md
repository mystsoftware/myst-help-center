## Use Case

Ever wanted to maintain an active/passive environment where your standby nodes are shutdown? You may want to benefit from cost saving in cloud infrastructure or as a corporate requirement for disaster recovery purposes.



Myst supports standby nodes which are inaccessible via an SSH connection. Continue reading for more information as there are some limitations to be mindful of.



## Enabling the feature

1. Go to **Platform Blueprint** > **Global Variables**
2. Create a new global variable:
   `validation.ssh=false`
3. Run the **update** action *making sure all your nodes are initially active.*



## Myst Action List

| Product      | Action         | Standby Node State  | Outcome | Notes                                                        |
| ------------ | -------------- | ------------------- | ------- | ------------------------------------------------------------ |
| FMW          | provision      | Active              | Success | All nodes are required to be active                          |
| FMW          | restart        | Down                | Success | Use the **limit nodes** feature to ignore `standby nodes`    |
| FMW          | patch          | Down                | Success | Use the **limit nodes** feature to ignore `standby nodes`.<br />Best practice to start stand by and apply patches in a rolling fashion |
| FMW          | update         | Down                | Success | Configuration changes are applied to the AdminServer.<br /><br />JCA Adapter configuration **Distributed Mode** should be ideally be set to `shared`. If set to `distributed` then only the active nodes are updated. When the standby nodes come online then the 'update' action needs to be run. |
| FMW          | start          | Down back to Active | Success | Code synchronizes from AdminServer allowing FMW to come up in a compatible state |
| OSB / SOA    | deploy         | Down                | Success | Code synchronizes when standby nodes become active           |
| Custom XPath | deploy-xpath   | Down                | Failure | Cannot connect to standby nodes to deploy the JAR library    |
| Jar Library  | deploy-library | Down                | Failure | Cannot connect to standby nodes to deploy the OSB Custom XPath library |

