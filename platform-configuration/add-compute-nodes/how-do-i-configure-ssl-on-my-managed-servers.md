# How do I configure SSL on my Managed Servers?

### Prerequistes
The following must be done before following this article:
1. You must have created an identity store using keytool for any certificates would need to use for the configuration of SSL on WebLogic Server
2. You must have created a trust store using keytool for all of the Certificate Authorities you wish to trust, including the ones you used to sign the identity certificates from step 1.
3. These Java Keystores must be installed on the targets either in a shared location or copied to each server individually.


### Configuring the Keystores
The keystores are a secure way of storing the SSL identities of your servers. You configure MyST there using the Blueprint (as they tend to be common across your domain). It is not uncommon to configure four key stores, two each for your non-production environment and two each for your production environments. 

To configure your keystores select `Modeling > Platform Blueprints` and select your platform blueprint. Select `Edit Configuration` and click the `+` next to the `Keystores` option in the blueprint, this will create a new Keystore node. Clicking on `Keystore - 1` will allow you to configure this as show below:
![](/assets/KeyStore Screen.png)

Give the `Component` a name to easily identify this keystore. In the `Location` enter the location of the keystore on the target servers. This should use a variable from the model to ensure this is consistently applied, such as the SOA Shared directory as we have done below. The `Password` is the keystore password which was used to protect the store as a whole. Repeat the above configuration for any additional keystores you need, in general you will need at least two. An example of this is shown below:
![](/assets/acme JKS config.png).

### Configure the Admin Server
The Admin server can be configured in the Blueprint, but the Managed Servers must be configured in the models (as the model can have a different number of server).

To configure the SSL on the Admin server, select the Blueprint as above and Edit. Select the `WebLogic Domains > [domain name] > Admin Server`. Click the `Edit` button to edit the configuration. Enter value per the table below.
