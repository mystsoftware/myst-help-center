MyST provides an intuitive data model to define your Fusion Middleware model that covers most of the configuration you need to set up. However, you can extend MyST to achieve specific configuration requirements that are not available in the model.

Once the extensions for the resources have been setup, you can also remove them from the MySt model, where you have defined the extensions.

**NOTE:** Define declarative WLST only in the MyST model that has the configuration you want to remove. Contact [Rubicon Red Support](mailto:myst.support@rubiconred.com) if you need further information.



### To remove WebLogic resources configured in MyST:

As the process of 'Generating a MyST extension' is completed, you can remove the extension you have generated, in the same way you have defined it inside the model. Consider the extension you want to remove and use it's attribute for its removal.

Please use the below configuration format to remove the extension defined..

**<attribute name="_present">0</attribute>**

![1566310875553](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1566310875553.png)

