## {{page.title}}

MyST will introspect WebLogic domains so that they can be brought under the management of MyST. Occasionally, you may discover an unconventional domain which can not be immediately managed by MyST even after the introspection. This is usually caused by a domain that has been setup with undesirable hacks which would make it heavily deviate from a standard archetype of a WebLogic domain. This article is for tracking instances where additional tweaks are required before an introspected domain can be functional. 

### Unconventional 12c dual machines with one running node manager

#### Background

Since 12c, it has been necessary for there to be a separate node manager for the admin server and managed server if they are deployed to the same host. In 11g, you could have just the one Node Manager for both.

#### Problem

A customer has followed an unconventional solution to make the Admin Server  node manager configured on `localhost:5556` and the Managed Server node manager configured on `<actual hostname>:5556`. This trick WebLogic into thinking there are the node manager when there is only one. This would never be recommend this but if it has already been done, MyST needs to provide a way to manage this unconventional environment.

#### Solution

If you cannot simply re-build the environment to follow the correct standards for 12c and have to keep it with the unconventional standard. Then, before you can manage it with MyST, you would need to [introspect the environment using MyST](https://docs.rubiconred.com/myst-studio/platform/introspection/) and then afterwards make some changes to the Platform Blueprint. The steps to address this are described in the workaround below.

#### End-to-End Example

After introspection you may have an extra compute group and an extra compute node because of the 'localhost' hack. You're blueprint and model would look similar to the diagrams below.
 
**Blueprint**
![](/assets/pastedImage_1.png)

**Model**
![](/assets/pastedImage_4.png)
 
First, make sure you are on MyST 5.8.1+
Next, using the UI we can remove the extra compute group from the blueprint and the extra compute node from the model as shown below.
 
**Model**
![](/assets/pastedImage_6.png)
 
**Blueprint**
![](/assets/pastedImage_7.png)
 
After this, we have to target the Admin Server product to the other compute group and node. This is possible to do from 5.8.1+.
 
**Blueprint**
![](/assets/pastedImage_10.png) 
 
**Model**
![](/assets/pastedImage_13.png)
 
Now all that remains is to be able to set the node manager listen address and listen port for the Admin server machine according to the unconventional `localhost` approach. That can be done by editing the model.
![](/assets/pastedImage_14.png)

The topology should now be in the desired state so that you can use MyST to deploy to the platform instance. All that will be left to do is to choose the **Provision** action while selecting the pre-existing flag. This final step puts the Platform Instance into an `ACTIVE` state without needing to do an actual re-provisioning. Now you can manage your unconventional environment in style.

