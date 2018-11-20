

As part of the Enterprise Deployment Guide (EDG) Oracle recommend using a virtual IP (VIP) for the admin server. This article explains the steps to setup a VIP and failover the admin server (collocated).

There is no need to create a new compute node for the admin server. Compute nodes in MyST are represented as physical servers.

#####Assumptions

* Two physical servers
  * Physical1 - Managed Server1 + Admin Server (collocated)
  * Physical2 - Managed Server2
* Failover between existing nodes in the domain
* Shared storage for aserver domain home



## Setup Admin Server with a VIP
MyST uses the admin server machine's listen address to also set the admin server listen address. Here we set the machine listen address.

1. Use a VIP for the admin server machine's listen address
   ![](img/using-adminserver-with-vip-and-failing-over-01.png)

## Failing Over the Admin Server

Configuration change sin MyST are required to fail over the admin server to another node in the domain. 

#### Updating Compute Nodes

1. Go to **Compute Nodes**
2. Update the **Admin Server** product to it's respective node. For example:
  1. Delete from node 1![](img/using-adminserver-with-vip-and-failing-over-02.png)
  2. Add into node 2![](img/using-adminserver-with-vip-and-failing-over-03-1.png)
3. Click ![](img/using-adminserver-with-vip-and-failing-over-03-2.png) to refresh the data in MyST
4. Go to **Machines**
5. Remove the auto-computed machine
   ![](img/using-adminserver-with-vip-and-failing-over-04.png)
6. Restore the **Name** and **Listen Address**. This was re-computed by MyST when updating the Admin Server product.
  ![](img/using-adminserver-with-vip-and-failing-over-05-1.png)
7. Click ![](img/using-adminserver-with-vip-and-failing-over-05-2.png)



#### Updating the Platform Instance Revision

Force MyST to update the platform model [pmNN] revision (which contains the above compute node changes) you can use the reprovision option while selecting 'Environment already pre-provisioned' checkbox.

1. Go to Actions > **Reprovision**
![](img/using-adminserver-with-vip-and-failing-over-06-1.png)
2. Select **Environment already pre-provisioned?**
![](img/using-adminserver-with-vip-and-failing-over-06-2.png)
3. Click ![](img/using-adminserver-with-vip-and-failing-over-06-3.png)



## MyST Actions

The MyST actions are expected to work as normal. No changes are required.

#####Patch

* MyST runs patch action against each compute node

#####Install

* Product binaries are installed to each compute node

#####Update

* The update action connects to the admin server's listen address (VIP) using WLST