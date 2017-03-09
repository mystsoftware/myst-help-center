You can either specify a SQL file in your workspace (using Administration > Custom Actions), or have the SQL files bundled and published to your binary repository (recommended).

The four available phases you can use combined with RCU:
* rcu-create-pre
* rcu-create-post
* rcu-drop-pre
* rcu-drop-post

###Prerequisites
####Preparing the SQL Files
#####Workspace
1. Go to Administration > Custom Actions
1. Create a sql script in the workspace
1. ![WebLogic Record](/platform-configuration/sql-scripts-pre-and-post-rcu/custom_actions.png)

#####Binary Repository
1. Create a ZIP file and publish to your binary repository
1. Ensure that you have a MyST Studio Continuous Delivery Profile defined that is pointing to your binary repository

###Configuring MyST Studio
####Workspace
If you use the **Workspace** method:
1. Define the SQL deployment definition in the global properties section
```
core.deployment[Stock-rcu-create-post].param[data-source-username]=sys
core.deployment[Stock-rcu-create-post].param[data-source-sys-role]=sysdba
core.deployment[Stock-rcu-create-post].param[data-source-password]=${core.product[rcu].param[db-password]}
core.deployment[Stock-rcu-create-post].param[data-source-url]=${core.product[rcu].param[db-url]}
core.deployment[Stock-rcu-create-post].param[execute]=resources/custom/sql/myst-rcu-post.sql
core.deployment[Stock-rcu-create-post].param[fail-on-error]=false
core.deployment[Stock-rcu-create-post].param[phase]=rcu-create-post
core.deployment[Stock-rcu-create-post].present=true
core.deployment[Stock-rcu-create-post].type=sql
```

####Binary Repository
If you use the **Binary Repository** method:
1. Define the SQL deployment definition in the global properties section

```
core.deployment[Stock-rcu-create-post].artifact.repository.artifactId=myst-rcu-post
core.deployment[Stock-rcu-create-post].artifact.repository.groupId=com.rubiconred.myst
core.deployment[Stock-rcu-create-post].artifact.repository.type=zip
core.deployment[Stock-rcu-create-post].artifact.repository.version=1.0
core.deployment[Stock-rcu-create-post].param[data-source-username]=sys
core.deployment[Stock-rcu-create-post].param[data-source-sys-role]=sysdba
core.deployment[Stock-rcu-create-post].param[data-source-password]=${core.product[rcu].param[db-password]}
core.deployment[Stock-rcu-create-post].param[data-source-url]=${core.product[rcu].param[db-url]}
core.deployment[Stock-rcu-create-post].param[execute]=(EMBEDDED)/rcu_create_post.sql
core.deployment[Stock-rcu-create-post].param[fail-on-error]=false
core.deployment[Stock-rcu-create-post].param[phase]=rcu-create-post
core.deployment[Stock-rcu-create-pre].present=true
core.deployment[Stock-rcu-create-post].type=sql
```

###Provisioning
When you run a provision (or RCU) action and depending on which phase you are using MyST will also execute the SQL.