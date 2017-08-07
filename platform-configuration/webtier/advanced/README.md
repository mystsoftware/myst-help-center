MyST has out of the box support for customising webtier routing rules and standard settings as described [here](https://docs.rubiconred.com/myst-studio/platform/resources/weblogic/servers/web-tier.html). This approach takes the inputs from attributes defined in the MyST Platform Definition and uses these to generate the `moduleconf\*.conf` files and `httpd.conf`, `admin.conf` and `sshd.conf`. But what if you want full control over the files being created? 

In this case, you may find it more fitting to define a template file for replacement when the webtier is configured at provision-time. 

### MyST Webtier Template - A working example

Let's say for instance, that you wanted to create a file called `moduleconf/custom.conf` with the following:

```
<Location /customrout>
SetHandler weblogic-handler
WeblogicCluster osb02.acme:9011,osb02.acme:9011
DynamicServerList ON
RequestHeader set REQUESTORDN "%{SSL_CLIENT_S_DN}s"
Debug ERR
WLLogFile /u01/app/oracle/user_projects/instances/ohs_instance_osb_domain/diagnostics/logs/OHS/osb_domain/wlproxy_advies.log
DebugConfigInfo OFF
WLTempDir /tmp/_wl_proxy_12c
</Location>
```

Whilst the above could be achieved entirely through the [Routing Rules](https://docs.rubiconred.com/myst-studio/platform/resources/weblogic/servers/web-tier.html) construct in the MyST Platform Definition, it may be easier to use a template. Let's look at the templated version of our `custom.conf`.

```
<Location /customroute>
SetHandler weblogic-handler
WeblogicCluster ${core.domain.cluster[osb].cluster-address}
DynamicServerList ON
RequestHeader set REQUESTORDN "%{SSL_CLIENT_S_DN}s"
Debug ERR
WLLogFile /u01/app/oracle/user_projects/instances/ohs_instance_${core.domain.name}/diagnostics/logs/OHS/${core.domain.name}/wlproxy_advies.log
DebugConfigInfo OFF
WLTempDir /tmp/_wl_proxy_12c
</Location>
```

Notice that we have replaced `osb_domain` with `${core.domain.name}` so that it takes the value from the MyST Platform Definition. Similarly, we have replaced `osb02.acme:9011,osb02.acme:9011` with `${core.domain.cluster[osb].cluster-address}`. This ensures that the template can be reusable across multiple MyST Platform Instances.

### Registering our Webtier Template in MyST Studio

We can register our template with MyST as follows:

1. Click **Administration** > **Custom Actions** > **Create New**
2. Fill in the mandatory/optional fields. Be sure to choose `Plain text` for the **Resource Type** and take note of the **Target Location**, we will provide this value to our Platform Blueprint later. The name is for display only, choose something that will make it clear what the action is to be used for.
![](/assets/Screenshot 2017-08-07 11.29.05.png)
3. Click Save.

The name of the file can be anything as long as `.conf` is the file extension. Oracle Webtier will dynamically discover any files under `moduleconf` with this extension and MyST will copy any files to `moduleconf` that exist in the source directory which we indicate in the next step.

### Updating our Platform Blueprint to use the Webtier template

1. In the Platform Blueprint editor, navigate to **Webtier Configuration**. Enter the value of **Moduleconf Source** > **Directory** to `${myst.workspace}/resources/custom/ohs`
2. Save the changes
3. During a provision, the custom webtier routing rules will be taken from the Webtier template.

To apply the changes without doing a reprovision:
1. Run `configure-webtier` as a custom action
2. Restart all servers
3. Notice that any `moduleconf` files have been replaced and automatically copied to all servers.

### What about httpd.conf, ssh.conf and admin.conf?

Performing advanced customisations to `httpd.conf`, `admin.conf` and `sshd.conf` is not as straight forward as defining custom routing rules under `moduleconf`. For this reason, it is preferable to define all customisation via `moduleconf`. In an event this approach is not feasible, it would be possible to customise these files using the combination of a template and a custom action which executes the template customisation.
To update these files you would need to 
1. create a template and register it as a custom action (e.g. `Plain text` type at `resources/custom/httpd/httpd.conf`)
2. create a custom action (e.g. `update-webtier-httpd-conf`) which programmatically takes the template and writes it to the desired location
3. reference the custom action after `configure-webtier` (e.g. `action.configure-webtier.post=update-webtier-httpd-conf`)

Below is an example **Jython Action** for replacing `httpd.conf` with a template 

```
from com.rubiconred.myst.config import ConfigUtils
import os

def myst(cfg):
    node_list = webtier.getNodeList()
    if node_list:
        nodes = node_list.getNodeArray()
        for node in nodes:
            ohs_node = node.getId()
            cfg.setProperty("ohs.instance.host",cfg["core.node["+ohs_node+"].host"])
            cfg.setProperty("ohs.component.name",cfg["core.webtier.node["+ohs_node+"].component-name"])
            cfg.setProperty("ohs.instance.home",cfg["core.webtier.node["+ohs_node+"].instance-home"])
            os.system("cp resources/custom/httpd/httpd.conf "+cfg['ohs.instance.home']+"/httpd.conf")
            ConfigUtils.findAndReplaceFile(File(cfg['ohs.instance.home']+"/httpd.conf"), cfg.configuration.getProperties(), false)
```