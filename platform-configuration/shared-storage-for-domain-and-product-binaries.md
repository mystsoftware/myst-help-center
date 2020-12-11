## Use Case

After provisioning a domain with the **maximum size of nodes** are you scaling nodes down or up by utilising shared storage for aserver/mserver domain and product binaries directories?

*NOTE: Use in conjunction with [limiting SSH validation on standby nodes](platform-configuration/limiting-ssh-validation-for-standby-nodes) feature to stop Myst performing SSH validations against scaled down nodes *



## Configuring the Myst Blueprint/Model

#### Enabling the Feature

1. Go to **Platform Blueprint** > **Global Variables**
2. Create a new global variables:
   `validation.multi-node.install=false`
   `validation.ssh=false`
3. Create the Platform Model with your **maximum size of nodes**. You can scale down from the maximum and and up to this maximum but no higher.
4. Run the **provision** action with all nodes active.



#### Updating the NodeManager

###### AdminServer

```shell
-DDomainsFile=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/nodemanager.domains -DNodeManagerHome=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/soa-as -DLogFile=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/soa-as/nodemanager.log
```

###### Managed Server

```shell
-DDomainsFile=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/nodemanager.domains -DNodeManagerHome=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/soa-01 -DLogFile=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/soa-01/nodemanager.log
```



#### Updating the oraInventory Directory

Because the product binaries are installed in shared storage we want the relating oraInventory to also be there.

1. Go to **Platform Blueprint** > **Global Variables**
2. Update **oui.inventory.directory** to: `${oracle.base}/admin/oraInventory`



## Myst Action List

Information about the updated actions which are supported in Myst `6.6.3+`.

| Name        | Standard Feature                                             | Shared Storage Feature                                       |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| install     | Install product binaries on all nodes                        | Installs product binaries one time on the AdminServer node into the shared storage |
| patch       | OPatch on all nodes                                          | Runs on the AdminServer node. Opatch is applied one time on shared storage product binaries |
| copy-domain | 1. Packs domain aserver<br />2. Unpacks domain to mserver nodes | 1. Packs domain aserver<br />2. Unpacks the domain to the first mserver node |

