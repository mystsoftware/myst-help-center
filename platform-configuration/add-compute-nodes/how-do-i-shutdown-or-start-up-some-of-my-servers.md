## Start up or shutdown of some of your Servers

With MyST Studio it is possible to start up or shutdown servers the following ways:

* All servers in the domain
* All servers on a specific compute node
* Specific servers no matter the node

The first of these two are covered in MyST Studio documention. This article shows how you can perform these life-cycle events on specific servers within the cluster.

Select `Provisioning > Plaform Instances` and select the platform model you wish to manage the life cycle of. Select the `Actions` button on the top right and click the `Control` option. You will see something like the following screen.
 
![](/assets/Actions Control.png)

Instead of performing the default Start action, select the dropdown below the `Select an action to perform.` and select the `Custom` option. The custom option allows you perform any MyST action on the platform. In this case we'll use the start-via-as or the stop-via-as actions for starting or stopping servers respectively. The *-via-as is a MyST command which will use the admin server (as opposed to the node manager for example) to start or stop the servers. Enter `stop-via-as` or `start-via-as` in the `Action Name` text field.

In the `Additional Arguments` text box we need to provide the servers we wish to perform this action on. For example if we had to two SOA servers in the cluster and we wanted to stop them we could enter `-Dservers=soa.server1,soa.server2`. An example of this can be seen below:
![](/assets/Actions Control - start-via-as.png)

Clicking `Execute` will perform this action like any other.

**Note**: Other example of the servers could be osb.server[n] or wsm.server[n] etc...