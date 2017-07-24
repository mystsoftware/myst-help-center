The MyST CLI is used by MyST Studio behind-the-scenes to execute operations against target environments. However, the MyST CLI can also be used to trigger operations within MyST Studio. This can be useful in some cases such as:
 * When you require the execution of MyST to be performed from a **third party tool** such as Jenkins, Puppet or Chef but wish to maintain the change audit history and platform configuration details within MyST Studio
 * When you are already using MyST CLI with Jenkins (or another CI server / configuration management tool) and want to **phase the migration to MyST Studio**. For instance, you may wish to continue to use Jenkins as the frontend in the short term while the configuration data is migrated to Studio.
 * When you have a **complex deployment process** and require ultimate control over the artifact deployment ordering sequence and how it interoperates with other deployment steps that sit outside MyST.

## One Time Configuration Steps

### Retrieve API key from MyST Studio console 

1. Login to MyST Studio with an administrator account
2. Click on "Administration" then select "Users"
3. Under the MySTAdministrator \(API User\) click on drop-down and select "Show API Key"
![](img/1.show-api-key.png)
4. Copy the key, we will use this later in our MyST workspace. If you want, you can generate a new key at any time.
![](img/2.api-key-view.png)

### Create and version control the MyST Workspace

MyST CLI has the concept of a MyST workspace which is a metadata directory that can be version controlled. In this example we will use Git.

For non-Studio customers, the MyST workspace is used for storing Platform Blueprint and Models in XML or Properties format. For Studio customers, the Platform Blueprints and Models are version-controlled in the MyST Studio repository. Platform Blueprints and Models can be retrieved from MyST Studio by defining MyST Studio connectivity details within the MyST workspace. Common use cases for this approach are:

* Creating CI server jobs that perform operations using configuration data from MyST Studio
* Using a hybrid of configuration and deployment data from a version control system like Git and MyST Studio \(e.g. Secure platform connectivity details are stored in MyST Studio but the deployment data is stored on the file system in the MyST workspace\)
* Creating MyST custom action extensions which use configuration data from MyST Studio.

If you don't have an existing MyST workspace, you can create and version control one as follows:

1 . Create a new folder for our MyST workspace.
```
mkdir myst-workspace
```
2 . Navigate to the myst-workspace and initialize it using MyST

```
cd myst-workspace
myst init
```

This will provide an output similar to the following:

```
MYST_HOME=/opt/myst
MYST_WORKSPACE=/u01/app/oracle/admin/myst-workspace

------------------------------------------------------------------------
R U B I C O N >< R E D MyST v3.6.2.1 (149)
...delivery...deMySTified..............................................
(c) 2011-2016 Rubicon Red Software Pty Ltd. All rights reserved
------------------------------------------------------------------------
Registered to : EMAILADDRESS=support@rubiconred.com, CN=Rubicon Red, L=Brisbane, ST=QLD, C=Brisbane
Expires : Tue Aug 20 02:44:07 EDT 2019
Session ID : 05-04-16-22-14-15-8134
------------------------------------------------------------------------

Analysing request...

MyST workspace created at /u01/app/oracle/admin/myst-workspace

SUCCESS - 1 Second
```

3 . Create a file relative to the myst-workspace at `resources/myst.properties`

Add the details for your MyST Studio instance and include the API key that you copied before:

```
myst.studio.hostname=<MYST STUDIO HOST>
myst.studio.port=8085
myst.studio.api.key=<API KEY>
```

4 . Run the following command to display the myst usage

```
myst usage
```

This should retrieve all of the available Platform Models from MyST Studio. These platform models are named in the following convention:

`env.<environment type>.<platform model>.platform`

For example:

```
Configuration          Description
-------------          -----------
env.ci.soa.platform    CI SOA Environment
env.dev.soa.platform   Development SOA Environment
env.uat.soa.platform   User Acceptance Testing Environment
env.prod.soa.platform  Production SOA Environment
```

Take note of the Platform Model which you want to use.

5 . You can execute the MyST properties action on your platform model of choice by performing the following:

```
myst properties env.prod.soa.platform
```

This will display all of the properties of the given Platform Model.

6 . Lastly, version control your MyST workspace using your Version Control System of choice. In this example, we are using Git

```
git add myst-workspace/conf
git add myst-workspace/resources
git commit -m "Added MyST workspace"
git push origin master
```

> NOTE: From the previous execution there will be a logs directory created under myst-workspace. You should avoid version controlling this file by following the steps above to only version control the conf and resources directories.

### Execute actions using the MyST Studio data

To see a full list of MyST actions, you can execute 
```myst usage``` from the MyST workspace.

MyST actions executed from the MyST CLI with MyST Studio data follow the standard below:
```
myst <action name> env.<environment type>.<platform model>.platform
```