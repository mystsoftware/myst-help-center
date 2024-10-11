In MyST 5.5.1.0+ new Platform Blueprints and Models prompt for the Database **Host**, **Port **and **Service Name** rather than the complete URL. Existing Platform Blueprints and Models can be updated to utilize Database **Host**, **Port **and **Service Name** properties. This functionality allows support for short format JDBC URLs to be used with Oracle RCU and long format JDBC URLs with WebLogic JDBC DataSources such as GridLink.

### Understanding the New RCU Properties
#### Platform Blueprint
The following properties have been added and updated:
* db-url
  * set as a short url
  * uses `db-host`, `db-port` and `db-service-name` properties (more on this in the Platform Model section)
* db-runtime-url
  * set as a short url if using generic DataSources or;
  * set as a long url if using gridlink DataSources
* the default OFMW platform DataSources (eg. mds-soa) use the `db-runtime-url` property
  <br> ![WebLogic Record](/platform-configuration/upgrading-to-v5510+-new-rcu-properties/blueprint_db_url.png)

#### Platform Model
The following properties are new and will be **undefined**:
* db-host
* db-port
* db-service-name

### Using the new RCU Properties
To effectively use the new RCU properties you can:
1. Go to your Platform Model > Product > RCU
2. Update the `db-host`, `db-port` and `db-service-name` properties
 <br> ![WebLogic Record](/platform-configuration/upgrading-to-v5510+-new-rcu-properties/new_db_props.png)
3. Delete the `db-url`. This resets the property so it now inherits from the blueprint.
 <br> ![WebLogic Record](/platform-configuration/upgrading-to-v5510+-new-rcu-properties/delete_db_url.png)
4. Click Save and Save again.
5. You will see the `db-url` re-generated with a resolved value.
 <br> ![WebLogic Record](/platform-configuration/upgrading-to-v5510+-new-rcu-properties/delete_db_url2.png)
6. Save and Commit your changes

