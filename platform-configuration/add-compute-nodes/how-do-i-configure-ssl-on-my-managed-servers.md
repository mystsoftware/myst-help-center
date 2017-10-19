# How do I configure SSL on my Domain?

### Prerequistes

The following must be done before following this article:  
1. You must have created an identity store using keytool for any certificates you need to use for the configuration of SSL on WebLogic Server  
2. You must have created a trust store using keytool for all of the Certificate Authorities you wish to trust, including the ones you used to sign the identity certificates from step 1.  
3. These Java Keystores must be installed on the target server either in a shared location or copied to each server individually.

### Configuring the Keystores

The keystores are a secure way of storing the SSL identities of your servers. You configure MyST using the Blueprint \(as they tend to be common across your domain\). It is not uncommon to configure four key stores, two each for your non-production environment and two each for your production environments.

To configure your keystores select `Modeling > Platform Blueprints` and select your platform blueprint. Select `Edit Configuration` and click the `+` next to the `Keystores` option in the blueprint, this will create a new Keystore node. Clicking on `Keystore - 1` will allow you to configure this as show below:  
![](/assets/KeyStore Screen.png)

Click the `Edit` button and give the `Component` a name to easily identify this keystore. In the `Location` enter the location of the keystore on the target servers. This should use a variable from the model to ensure this is consistently applied, such as the SOA Shared directory as we have done below. The `Password` is the keystore password which was used to protect the store as a whole. Repeat the above configuration for any additional keystores you need, in general you will need at least two. An example of this is shown below:  
![](/assets/acme JKS config.png).

### Configure the Admin Server

The Admin server can be configured in the Blueprint, but the Managed Servers must be configured in the models \(as the model can have a different number of servers\).

To configure the SSL on the Admin server, select the Blueprint as above and Edit. Select the `WebLogic Domains > [domain name] > Admin Server`. Click the `Edit` button to edit the configuration. Enter value per the table below.

| Configuration Name | Value | Notes |
| :--- | :--- | :--- |
| Custom Identity KeyStore | rxr.def.Keystore-1 | This refers to the first Keystore you configured in the above step |
| Custom Trust KeyStore | rxr.def.Keystore-2 | This refers to the second Keystore you configured in the above step |
| Ssl Configuration &gt; Enabled | True | Setting this to true will enable the SSL listen port on the server |
| Ssl Configuration &gt; Listen Port | &lt;Any Port&gt; | Set this to the port you wish the server to listen on. |
| Ssl Configuration &gt; Server Private Key Alias | &lt;Alias&gt; | This is the name of the alias in the Custom Identity KeyStore which will be used as the identity for this server |
| Ssl Configuration &gt; Server Private Key Password | &lt;Password&gt; | This is the password you used for the private key above. |

Save and commit you changes to the blueprint.

The following is an example of the SSL configuration:![](/assets/SSL configuration.png)

### Configure the Managed Servers

Configuring the manager servers is exactly the same as the Admin except it must be done in the models and needs to be done for each managed server you wish to configure SSL for.

### Applying your configuration

It is best to configure this before you provision your environment for the first time, then the provision can configure this the very first time. This can be done post provision and is done during a update of the platform.

**Note**: If applying SSL to the admin server subsequent to the provision due to a bug in MyST you must use this work-around:

* Apply the configuration to all but the Admin Node first
* Apply the configuration to the Admin server but set the Ssl Configuration &gt; Enabled" to false. This will configure the keystores but not enable the SSL listener.
* In the WebLogic console, enable the SSL listener manually
* Now set the Ssl Configuration &gt; Enabled to True



