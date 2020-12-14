## Background

After provisioning a domain with the *maximum sized nodes* are you scaling nodes down and up while utilising shared storage for aserver/mserver **domain** and product binaries directories?

*NOTE: Use in conjunction with [limiting SSH validation on standby nodes](platform-configuration/limiting-ssh-validation-for-standby-nodes) feature to prevent Myst performing SSH validations against scaled down nodes.*



## Infrastructure Requirements

Overview of the requirements to scale nodes up and down.

| Name                                              | Requirement Type      | Requirement Example                                          |
| ------------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| Node                                              | Physical/Virtual Host | Create the maximum number of nodes.                          |
| Node size                                         | Myst                  | Create the Platform Model with the maximum size of nodes.    |
| Enable feature                                    | Myst                  | [For configuration see **Enable the Feature** ](#enable-the-feature) |
| NodeManager                                       | Myst                  | [For configuration see **NodeManager**](#nodemanager)        |
| Deployment Plan Distribution                      | Myst                  | [For configuration see **Deployment Plan**](#deployment-plan) |
| oraInventory                                      | Myst                  | [For configuration see **oraInventory**](#orainventory-directory) |
| oraInventory                                      | Shared storage        | `/u01/app/oracle/admin/oraInventory/`                        |
| Product binary home<br />Both aserver and mserver | Shared storage        | `/u01/app/oracle/product/`                                   |
| Domain home<br />Both aserver and mserver         | Shared storage        | `/u01/app/oracle/admin/`                                     |



## Blueprint/Model Requirements

#### Enable the Feature

1. Go to **Platform Blueprint** > **Global Variables**
2. Create a new global variables:
   `validation.multi-node.install=false`
   `validation.ssh=false`
3. Create the Platform Model with your **maximum size of nodes**. You can scale down from the maximum and and up to this maximum but no higher.
4. Run the **provision** action with all nodes active.



#### NodeManager

The NodeManager normally runs on each node under the same NodeManager home directory eg. `${[rxr.wls.Domain-1].domainAserverHome}/nodemanager`.

We want the NodeManagers' configuration files on shared storage by adding override java arguments. This starts the NodeManagers in their respective homes.

1. Go to **Platform Blueprint** > **WebLogic Domain Configuration** > **Machines**
2. Click **Show advanced properties**
3. Add the below properties to the respective NodeManagers

###### AdminServer

*Line breaks added for easier viewing. Remove line breaks when entering into Myst.*

```shell
-DDomainsFile=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/nodemanager.domains 
-DNodeManagerHome=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/soa-as 
-DLogFile=${[rxr.wls.Domain-1].domainAserverHome}/nodemanager/soa-as/nodemanager.log
```

###### Managed Server

* *Line breaks added for easier viewing. Remove line breaks when entering into Myst.*
* *Increment the **soa-01** directory for each node*

```shell
-DDomainsFile=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/nodemanager.domains 
-DNodeManagerHome=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/soa-01 
-DLogFile=${[rxr.wls.Domain-1].domainMserverHome}/nodemanager/soa-01/nodemanager.log
```

Here is an example of a two node environment.

```
└── shared_storage
    ├── aserver
    │   └── soa-as
    │       ├── nodemanager.log
    │       ├── nodemanager.log.lck
    │       ├── nodemanager.process.id
    │       ├── nodemanager.process.lck
    │       └── nodemanager.properties
    └── mserver
        ├── soa-01
        │   ├── nodemanager.log
        │   ├── nodemanager.log.lck
        │   ├── nodemanager.process.id
        │   ├── nodemanager.process.lck
        │   └── nodemanager.properties
        ├── soa-02
            ├── nodemanager.log
            ├── nodemanager.log.lck
            ├── nodemanager.process.id
            ├── nodemanager.process.lck
            └── nodemanager.properties
```



#### Deployment Plan

Update the Myst deployment plan distribution to `shared`. Myst now distributes JCA adapter plans once instead of distributing to multiple nodes.

1. Go to **Platform Blueprint** > **WebLogic Domain Configuration**
2. Update **Deployment Plan Distribution Mode** to: `shared`



#### oraInventory Directory

Because the product binaries are installed in shared storage we want the relating oraInventory to also be there.

1. Go to **Platform Blueprint** > **Global Variables**
2. Update **oui.inventory.directory** to: `${oracle.base}/admin/oraInventory`



## Myst Action List

Information about the updated actions which are supported in Myst `6.6.3+`.

| Name        | Standard Feature                                             | Shared Storage Feature                                       |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| install     | Install product binaries on all nodes                        | Installs product binaries one time on the AdminServer node into the shared storage |
| patch       | OPatch on all nodes                                          | Runs on the AdminServer node. Opatch is applied one time on shared storage product binaries |
| copy-domain | 1. Packs domain aserver<br />2. Unpacks domain to mserver nodes | 1. Packs domain aserver<br />2. Unpacks the domain to one mserver node defined in Myst |



## Scaling Nodes Up and Down

#### Scale Up

1. **Infrastructure**
   1. <span style="color:DodgerBlue">**Create**</span> the physical/virtual node using your preferred tool
   2. Update DNS to reflect the FQDN of the <span style="color:DodgerBlue">**created**</span> node
   3. <span style="color:DodgerBlue">**Add**</span> the node from the load balancer
2. **Start WebLogic Manager Server and NodeManager**
   1. systemd will auto <span style="color:DodgerBlue">**start**</span> services
   2. (Optional) Myst
      1. <span style="color:DodgerBlue">**Start**</span> the managed server and nodemanager
      2. Platform Model > Actions > Control
      3. Select the node to <span style="color:DodgerBlue">**Start**</span>



#### Scale Down

1. **Start WebLogic Manager Server and NodeManager**
   1. systemd will auto <span style="color:Tomato">**stop**</span> services
   2. (Optional) Myst
      1. <span style="color:Tomato">**Stop**</span> the managed server and nodemanager
      2. Platform Model > Actions > Control
      3. Select the node to <span style="color:Tomato">**Stop**</span>
2. **Infrastructure**
   1. <span style="color:Tomato">**Terminate**</span> the physical/virtual node using your preferred tool
   2. Update DNS to reflect the FQDN from the <span style="color:Tomato">**terminated**</span> node
   3. <span style="color:Tomato">**Remove**</span> the node from the load balancer



*Note: See the documentation about [limiting SSH validation for standby nodes](platform-configuration/limiting-ssh-validation-for-standby-nodes.md) for an understanding of potential limitations when scaling.*
