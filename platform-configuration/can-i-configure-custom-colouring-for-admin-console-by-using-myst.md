Yes, you can configure the custom colouring for admin console by using myst

### Prerequisites

- Create a `plain text` file in myst [custom action](https://help.mystsoftware.com/platform-configuration/configure-myst-custom-action#creating-custom-actions-in-myst-studio) 

  - name:console-style 

  - location:console-style.sh

  - content: 

    ```
    #!/bin/bash
    
    FMW_HOME=$1
    if [ -z "" ]; then
        FMW_HOME=/u01/app/oracle/product/fmw1221
    fi
    BORDER_COLOUR=$2
    if [ -z "$BORDER_COLOUR" ]; then
        BORDER_COLOUR="red"
    fi
    
    echo "Backing up general.css and console.css"
    CSS=$FMW_HOME/wlserver/server/lib/consoleapp/webapp/framework/skins/wlsconsole/css
    cp $CSS/general.css{,.`date +%Y%m%d-%H%M%S`}
    cp $CSS/console.css{,.`date +%Y%m%d-%H%M%S`}
    
    echo "Adding Production styling to console.css"
    cat <<EOT >> $CSS/console.css
    #product-brand-name:before {
        background-color: $BORDER_COLOUR;
        content: "PRODUCTION";
        font-size: large;
        margin-right: 20px;
        padding: 35px;
    }
    EOT
    
    echo "Adding border colour"
    sed -i "/background-color: #FFFFFF;/a \ \ \ \ border: 8px solid $BORDER_COLOUR;" $CSS/general.css
    
    exit 0
    
    ```

- Create a `Jython Action` in the myst custom action

  - name: console-style 
  
  - location: console-style.py
  
  - content: 
  
    ```
    from com.rubiconred.myst.util import SSHUtil
    
    """
    ####
    #Description
    ####
    Updates ofmw CSS
    Follows steps from
    https://david-kerwick.github.io/2014-03-31-customise-skin-of-weblogic-admin-console/
    ####
    #Dependencies
    ####
    Myst custom action created as plain text file:
    console-style.sh
    
    #!/bin/bash
    
    FMW_HOME=$1
    if [ -z "" ]; then
        FMW_HOME=/u01/app/oracle/product/fmw1221
    fi
    BORDER_COLOUR=$2
    if [ -z "$BORDER_COLOUR" ]; then
        BORDER_COLOUR="red"
    fi
    
    echo "Backing up general.css and console.css"
    CSS=$FMW_HOME/wlserver/server/lib/consoleapp/webapp/framework/skins/wlsconsole/css
    cp $CSS/general.css{,.`date +%Y%m%d-%H%M%S`}
    cp $CSS/console.css{,.`date +%Y%m%d-%H%M%S`}
    
    echo "Adding Production styling to console.css"
    cat <<EOT >> $CSS/console.css
    #product-brand-name:before {
        background-color: $BORDER_COLOUR;
        content: "PRODUCTION";
        font-size: large;
        margin-right: 20px;
        padding: 35px;
    }
    EOT
    
    echo "Adding border colour"
    sed -i "/background-color: #FFFFFF;/a \ \ \ \ border: 8px solid $BORDER_COLOUR;" $CSS/general.css
    
    exit 0
    """
    
    def myst(cfg):
        fmw_home = cfg.getProperty('core.fmw.home')
        border_colour = cfg.getProperty('custom.vf.border_colour')
        if border_colour is None:
            border_colour = ''
        
        # Copy console-style.sh to each node
        nodes = cfg.getProperty('nodes')
        for node in nodes.split(','):
            machineId = cfg.getProperty('core.domain.machine[' + node + '].node-id')
            nodeName = cfg.getProperty('core.node[' + machineId + '].name')
            LOG.info('Running on: ' + nodeName)
            LOG.info('    /tmp/console-style.sh ' + fmw_home + ' ' + border_colour)
            SSHUtil.copyFileToNode(machineId, cfg.getProperty('myst.workspace') + '/resources/custom/console-style.sh', '/tmp/', 1);
            SSHUtil.executeCommandOnNode(machineId, 'chmod u+x /tmp/console-style.sh && /tmp/console-style.sh ' + fmw_home + ' ' + border_colour, 0)
    
    ```



### Applying custom colouring for admin console

#### Step 1: Edit Platform Blueprint

1. Navigate to the Platform Blueprint which you want to change in MyST Studio \("Modeling" &gt; "Platform Blueprints"\)
2. Click "Edit Configuration"
3. Go to "Global Variables".
4. Click + 
5. Add **custom.vf.border_colour=?**

#### Step 2: Edit and update the Platform Model 

1. Go "Modeling" &gt; "Platform Models"
2. Select the Platform Model corresponding to the previously edited Platform Blueprint
3. Go to "Actions" and "View configuration "
4. Go to "Global Variables"
5. Update the  **custom.vf.border_colour**  variable based on the environment(ex: blue,red)
6. Click "Save and commit"

##### Step 3: Update the Platform Model and run custom action

1. Select the previously edited Platform Model 
2. Go to "Actions" and run `update` action
3. Go to "Actions" and select `custom` then choose `console-style`
4. Run `Execute`

##### 