## Summary

* [Introduction](README.md)
* [Supported Browsers](which-browsers-supported.md)
* [What is Myst?](what-is-myst.md)


## Troubleshooting

* [Maven 3.9.1+ blocks http during deployment](platform-configuration/troubleshooting/maven-default-http-blocker.md)
* [Troubleshoot failed Myst actions](platform-configuration/troubleshooting-update-or-provision-failures.md)
* [Cannot open a Platform Model or Blueprint](platform-configuration/troubleshooting/corrupt-blueprint-or-model.md)
* [SSH Authentication](platform-configuration/how-do-i-solve-authentication-errors-when-myst-connects-to-a-server.md)
* [Test SSH connectivity](myst-management/ssh-key-pair-connectivity-with-myst.md)
* [Introspection and 'Data integrity constraints'](platform-configuration/data-integrity-constraints/data-integrity-constraints-during-introspect-and-environment.md)
* [Error java.lang.NullPointerException](platform-configuration/javalangnullpointerexception/javalangnullpointerexception.md)
* [Myst UI hanging](myst-management/container-configuration/myst-fails-after-large-amount-of-logging.md)


## Myst Management

* [Myst performance management](myst-management/performance/README.md)
* [Myst SSH to a DMZ host via a Jumpbox/Bastion](myst-management/ssh-via-jumpbox.md)
* [Custom Myst database](myst-management/how-do-i-use-a-custom-myst-database.md)
* [Setup notifications for success/failure of Myst actions](myst-management/notifications/README.md)
* [Myst license](myst-management/myst-license.md)
* [Integrate Myst CLI with Studio](myst-management/myst-cli-with-myst-studio.md)
* [Uploading large files](myst-management/uploading-large-files.md)
* [Use an Internet Proxy](myst-management/how-do-i-setup-myst-and-associated-components-to-use-an-internet-proxy.md)
* [Upgrading to v5.5.1.0+](platform-configuration/upgrading-to-v5510+-new-rcu-properties.md)
* [Using a MySQL8 database](myst-management/mysql8/README.md)


## Platform Configuration

### Custom Scripting

* [Integrate my custom scripts with Myst](platform-configuration/configure-myst-custom-action.md)
* [Configure myst-extension](platform-configuration/configure-myst-extension.md)
* [JDK version error](platform-configuration/jdk-issue.md)

### Introspection

* [Managing SOA CS domains with Myst](platform-configuration/introspection/soacs/README.md)
* [Add more nodes with Myst](platform-configuration/add-compute-nodes/README.md)
* [Bring existing environments into Myst](platform-configuration/i-have-existing-environments-i-built-outside-of-myst-how-can-i-bring-them-into-the-control-of-myst.md)
* [Bring oddball domains into Myst](platform-configuration/introspection/odd-domain/README.md)

### Operations

* [Clone Platform Blueprints/Models](platform-configuration/clone-blueprint-model/README.md)
* [Compare Platform Blueprints/Models](platform-configuration/How-to-compare-changes-between-two-versions-of-Models-Blueprints.md)
* [Extra OFMW product parameters](platform-configuration/product-parameters.md)
* [OPatch one server at a time](platform-configuration/apply-rolling-patches.md)
* [Configure SOA MBeans](platform-configuration/can-i-configure-soa-mbeans-with-myst-studio.md)
* [Configure JCA Adapters](platform-configuration/configure-jca-adapters.md)
* [Configure EJB Transaction Timeout](platform-configuration/add-compute-nodes/can-i-configure-ejb-transaction-timeout-through-myst.md)
* [Run SQL scripts pre/post RCU](platform-configuration/sql-scripts-pre-and-post-rcu.md)
* [Deleting Weblogic resources](platform-configuration/Removing-WL-resources-in-MyST.md)

### Provisioning Guides

* [Oracle Analytics Server](platform-configuration/oas/README.md)
* [Axway API Gateway](platform-configuration/axway/README.md)
* [Oracle Data Integrator (ODI) 12c](platform-configuration/provisioning-oracle-data-integrator-12c.md)
* [Vanilla Weblogic Domain](platform-configuration/Plain-Vanilla-Weblogic-Domain.md)
* [Oracle Order and Service Management 12c](platform-configuration/add-compute-nodes/provisioning-oracle-order-and-service-management-12c.md)

### Rest API

* [Manipulate Blueprint and Model payload](platform-configuration/rest-api.md)
* [Apply common settings across Blueprints](platform-configuration/rest-api-bulk-edit-blueprints.md)
* [Automate creation of resources with Jenkins](platform-configuration/creating-myst-studio-resources-using-rest-apis-with-help-of-jenkins.md)

### Scaling

* [Scaling nodes with shared storage for domain/binaries](platform-configuration/scaling-nodes-with-shared-storage-domain-and-product.md)
* [Configure AdminServer VIP for failover](platform-configuration/using-adminserver-with-vip-and-failing-over.md)
* [Limit SSH validation for standby nodes in DR](platform-configuration/limiting-ssh-validation-for-standby-nodes.md)
* [Systemd auto start](platform-configuration/can-i-automatically-start-ofmw-servers-when-linux-host-reboots-by-using-myst.md)
* [How do I shutdown/start Managed Servers](platform-configuration/add-compute-nodes/how-do-i-shutdown-or-start-up-some-of-my-servers.md)

### SSL

* [Update SSL Certificates for Myst](myst-management/myst-ssl/README.md)
* [Enable SSL (HTTPS) for Artifactory](myst-management/artifactory-ssl/README.md)
* [Enable SSL (HTTPS) for Jenkins](myst-management/jenkins-ssl/README.md)
* [Create KSS keystores using Myst](platform-configuration/can-i-create-kss-keystore-by-using-myst.md)
* [Configure SSL for WebLogic](platform-configuration/add-compute-nodes/how-do-i-configure-ssl-on-my-managed-servers.md)

### Webtier

* [Copy custom Webtier config and soft restart](platform-configuration/webtier/How-do-I-use-MyST-to-copy-custom-webtier-configuration-and-perform-soft-restart.md)
* [Custom HTML coloring for Admin Consoles](platform-configuration/can-i-configure-custom-colouring-for-admin-console-by-using-myst.md)
* [Advanced Webtier customizations](platform-configuration/webtier/advanced/README.md)


## Application

### Builds

* [Jenkins maven project build error](artifact-build/maven-build-issues.md)
* [Upgrade Maven builds from 11g to 12c](artifact-build/maven-11g-to-12c.md)

### Deployment SDK

* [Deploy ESS with Myst Deployment SDK](myst-apis-and-sdk/deploying-ess-using-deployment-sdk.md)

### Deployments

* [Deploy time customizations for an OSB project](application-deployment/osb-change-transport-details.md)
* [12c SQL*Plus deployments](platform-configuration/deploy-sql-12c-using-sqlplus.md)
* [Deploy artifacts during provisioning](platform-configuration/system-artifacts/README.md)
* [Jenkins Configure as Code](application-deployment/how-do-i-configure-JenkinsConfigurationasCode-for-Myst-Plugin.md)
* [Copy files to a host](application-deployment/how-do-i-copy-files-to-a-linux-host-using-myst.md)
* [Integrate existing deploy scripts](application-deployment/custom.md)
* [Test deployments in local](application-deployment/local/README.md)
* [Seed non-Oracle databases](application-deployment/other-databases.md)
* [Bulk import artifacts/application blueprints](application-deployment/can-i-do-a-bulk-import-of-artifacts-and-application-blueprints.md)
* [Bulk migrate to Artifact Property Registry](application-deployment/how-can-i-bulk-migrate-my-properties-to-be-available-in-the-artifact-property-registry.md)
* [Create SOA partitions with Myst](application-deployment/how-do-i-create-soa-partitions-with-myst.md)
* [Configuring BPM FlexFields](application-deployment/deploy-bpm-flexfields.md)
* [Skip application deploy during reprovisioning](application-deployment/clear-cache/README.md)
