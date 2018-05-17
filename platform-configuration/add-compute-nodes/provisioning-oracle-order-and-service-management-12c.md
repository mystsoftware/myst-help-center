## Provisioning Oracle Order and Service Management 7.5.3

### Preparing database privileges

In order to provision Oracle Order and Service Management \(OSM\) 7.5.3 successfully, the database used by the environment must be prepared by executing the following SQL for the sys user:

`grant create any context to sys with admin option;`

### Defining required properties

In order to provision Oracle Order and Service Management \(OSM\) 7.5.3 successfully with MyST Studio, a number of properties must be defined for the OSM product in your configuration model.

To define the required properties:

1. Navigate to the model for OSM
2. Click Edit Configuration at the top-right of MyST Studio
3. Navigate to 'Oracle Order and Service Management' in the Product section of the model tree displayed on the left-hand side.

![](/assets/provisioning-osm-product-properties.png)

The required properties are:

| Property | Expected value |
| :--- | :--- |
| frontend-http-port | HTTP port used by front-end load balancer directing traffic to OSM. |
| db-default-tablespace | Default tablespace OSM tables will be installed in. |
| db-model-data-tablespace | Tablespace OSM model tables will be created in. |
| db-model-index-tablespace | Tablespace OSM model index tables will be created in. |
| db-order-data-tablespace | Tablespace OSM order data tables will be created in. |
| db-order-index-tablespace | Tablespace OSM order index tables will be created in. |
| db-temp-tablespace | Temporary tablespace used for OSM database operations. |
| frontend-host | Host where front-end load balancer is located |
| frontend-http-port | HTTP port used by front-end load balancer directing traffic to OSM. |
| frontend-https-port | HTTPS port used by front-end load balancer directing traffic to OSM. |
| notification-email-address | Email address for OSM administrator |
| notification-email-server | Email host used by OSM |
| timezone-offset | Configured timezone of database used by OSM, expressed in offset from GMT in seconds. |
| user-admin-password | OSM administrator password. |
| user-automation-password | OSM automation user \(osm-automation\) password. |
| user-internal-password | Password for core OSM user \(osm-internal\) |
| user-studio-password | Password for Design Studio administrator user. |



