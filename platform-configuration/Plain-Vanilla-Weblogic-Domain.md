**Provisioning Vanilla WebLogic in MyST Studio**

This documents provides steps to provision a plain WebLogic Domain of Managed Servers with no other product on it. The procedure explained below, can be replicated to create environments of similar kind.
 While creating a BluePrint, please follow the below specifications to provision successfully.



**Creation of Blueprint only with WebLogic**

1. Create a New Blueprint, and select ONLY WebLogic as a product.. and Click next

   ![](img/vanilla-weblogic-product.png)



2. Provide the ‘number of nodes’ as per requirement & Click Next

   ![](img/vanilla-weblogic-component.png)

   

3. Create your Blueprint by defining other configurations as per your requirement & Click Finish.

   Now, you will have a Blueprint which contains ONLY Weblogic Product on it.

   

**Create & Configure a Model**

In our example, we have created a Weblogic Domain which has just the Weblogic server. We should manually add configuration for the server.Now create a New Model and Provision it with the below specifications…

Add a new Machine and Click SAVE (You’ll already see a Machine created by-default)

Go to the Model created >> WebLogic Domain Configuration >> Machines. 

Click Edit configuration, Click on ![1564576210744](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1564576210744.png)to create new machine. 

Provide name, compute node & listen address of the machine and Click OK.



![](img/vanilla-weblogic-machine.png)

Now create a CLUSTER by clicking the Add                                                    icon…

Go to the Model created >> WebLogic Domain Configuration >> Weblogic Clusters. 

Click Edit configuration, Click on     to create new cluster.

Provide name, compute group & cluster address of the cluster

![](img/vanilla-weblogic-cluster.png)

Once you create a Cluster with the above details, you’ll be able to see a Server created under Managed Servers (in Weblogic Domain Configuration)

·        Machine :: This lets MyST know on which node this server needs to be provisioned.

·        Listen Address :: Listen Address of Server

·        Listen Port Enabled :: If you want the Listen Port Enabled

·        Listen Port :: Port number of Server

 **Save and Commit the Platform Model. Now, you are ready to Provision/Re-provision a Domain.**





##### 																																			~ TO REMEMBER ~

###### *DONOT add Machines, Clusters in the Blueprint., but, rather it is to be done in the Model*

![](img/vanilla-weblogic-server.png)