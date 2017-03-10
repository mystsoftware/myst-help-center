MyST provides a flexible alternative to out-of-the-box MyST actions by allowing the use of custom actions. Administrators can create scripts which are tailored to their purposes.

###Prerequisites
####Create the WLST Script
Example provided.

```
def myst(cfg):
    """Display the Domain's Managed Server Names"""
    
    # Log to MyST INFO
    LOG.info('hello world')
    
    # Log to MyST DEBUG
    LOG.debug('hello debugged world')
    
    # Connect to Admin Server
    if 'admin.host' in cfg and 'admin.port' in cfg and 'core.fmw.admin.username' in cfg and 'core.fmw.admin.password' in cfg:
        LOG.info('Connecting to Admin Server')
        admin_server_url=cfg['admin.host'] + ':' + cfg['admin.port']
        connect(cfg['core.fmw.admin.username'],cfg['core.fmw.admin.password'],admin_server_url)
        domainRuntime()
        
        # Log the Managed Server names
        if 'servers' in cfg:
            servers=cfg['servers']
            serversList=servers.split(',')
            for server in serversList:
                LOG.info('Server Name: ' + cfg['core.domain.server[' + server + '].name'])
        disconnect()
```

####Create the Custom Action
1. Click Administrators > Custom Actions > Create New
1. Fill in the mandatory/optional fields
    * **NOTE** - 'Name' becomes the myst custom action name
![WebLogic Record](/platform-configuration/configure-myst-custom-action/create-custom-action.png)

1. Run the custom action against your platform instance
    * ![WebLogic Record](/platform-configuration/configure-myst-custom-action/run-custom-action.png)
