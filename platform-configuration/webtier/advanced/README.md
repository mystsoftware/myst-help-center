Myst has out of the box support for customising webtier routing rules and standard settings as described [here](https://docs.rubiconred.com/myst-studio/platform/resources/weblogic/servers/web-tier.html). This approach takes the inputs from attributes defined in the MyST Platform Definition and uses these to generate the `moduleconf\*.conf` files and `httpd.conf`, `admin.conf` and `sshd.conf`. But what if you want full control over the files being created?

In this case, you may find it more fitting to define a template file for replacement when the webtier is configured at provision-time.

### Myst Webtier Template - A working example

Let's say for instance, that you wanted to create a file called `moduleconf/custom.conf` with the following:

```
<Location /customroute>
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

Notice that we have replaced `osb_domain` with `${core.domain.name}` so that it takes the value from the Platform Model. Similarly, we have replaced `osb02.acme:9011,osb02.acme:9011` with `${core.domain.cluster[osb].cluster-address}`. This ensures that the template can be reusable across multiple Platform Instances.

### Registering the Webtier Template in Myst Studio

We can register our template:

1. Click **Administration** &gt; **Custom Actions** &gt; **Create New**
2. **Name**: `webtier-custom-router` but this can be anything you want
3. **Resource Type**: `Plain text`
4. **Target Location**: `ohs/custom.conf` which we will provide this value to our Platform Blueprint later
   ![](/assets/Screenshot 2017-08-07 11.29.05.png)
5. Click Save.

*NOTE:* Oracle Webtier will dynamically discover any files under `moduleconf` with this extension and Myst will copy any files to `moduleconf` that exist in the source directory.

### Updating the Platform Blueprint to use the Webtier Template

1. Go to the **Platform Blueprint** > **Webtier Configuration**.
2. **Moduleconf Source Directory**: `${myst.workspace}/resources/custom/ohs`
3. Save and Commit
4. During a provision, the custom webtier routing rules will be taken from the Webtier template

To apply the changes without doing a reprovision:  
1. Run `configure-webtier` as a custom action  
2. Restart all servers  
3. Notice that any `moduleconf` files have been replaced and automatically copied to all servers.

### What about httpd.conf, ssl.conf and admin.conf?

Performing advanced customisations to `httpd.conf`, `admin.conf` and `ssl.conf` is not as straight forward as defining custom routing rules under `moduleconf`. For this reason, it is preferable to define all customisation via `moduleconf`. In an event this approach is not feasible, it would be possible to customise these files using the combination of a template and a custom action which executes the template customisation.  

To update these files you would need to:

1. Create a template as a **custom action** and register as `Plain text` type at `resources/custom/ohs/httpd.conf`\)
2. Create a template as a **custom action** and register as `Plain text` type at `resources/custom/ohs/ssl.conf`\)
3. Create a template as a **custom action** and register as `Plain text` type at `resources/custom/ohs/admin.conf`\)
4. Create a **custom action** \(e.g. `webtier-update-conf`\) which programmatically takes the template and writes it to the desired location  
5. Create a **global variable** in the Platform Blueprint (e.g. `action.configure-webtier.post=webtier-update-conf`\)
6. Run `configure-webtier` as usual. This will result in `configure-webtier` running and then the below custom action `webtier-update-conf` **overwriting** the conf files with ones defined as `Plain text` type in the custom action.

Below is an example **Jython Action** for replacing `httpd.conf` | `ssl.conf` | `admin.conf` with a template

```python
from com.rubiconred.myst.config import ConfigUtils
from java.io import File
from java.util import Properties
import os

def myst(cfg):
  webtier = cfg.configuration.getModel().getCore().getWebtier()
  node_list = webtier.getNodeList()
  if node_list:
    nodes = node_list.getNodeArray()
    for node in nodes:
      ohs_node = node.getId()
      props = Properties()
      props.setProperty("ohs.instance.host",cfg["core.node["+ohs_node+"].host"])
      props.setProperty("ohs.component.name",cfg["core.webtier.node["+ohs_node+"].component-name"])
      props.setProperty("ohs.instance.home",cfg["core.webtier.node["+ohs_node+"].instance-home"])
      
      for file in ["httpd.conf","ssl.conf","admin.conf"]:
        #findAndReplaceFile(srcFile,properties,allowUnresolvedProperties)
        replacedFile = ConfigUtils.findAndReplaceFile(File("resources/custom/ohs/" + file), props, 1)

        LOG.info("Backup " + file + " with suffix .yyyymmdd-HHMM")
        os.system('cp -p' + cfg['ohs.instance.home'] + '/' + file + ' ' + cfg['ohs.instance.home'] + '/' + file + '.`date "+%Y%m%d-%H%M"`')

        LOG.info("Copying " + replacedFile.getAbsolutePath() " to " + cfg['ohs.instance.home'] + "/" + file)
        os.system("cp -p" + replacedFile.getAbsolutePath() + " " + cfg['ohs.instance.home'] + "/" + file)
        
        LOG.info("*****************************\n*****************************")
        LOG.info("Oracle OHS requires the AdminServer to be restarted in order for the conf files to be pushed to OHS nodes.")
        LOG.info("*****************************\n*****************************")

```

