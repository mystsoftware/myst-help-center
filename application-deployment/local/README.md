# {{page.title}}

In some cases, you may wish to test out deployment using MyST from a local development environment. This is useful for ensuring automated deployment will work with MyST prior to committing the change to a version control system.

To perform local deployment, you will need to:
1. Install the MyST CLI
2. Create a `myst-deploy.sh` wrapper script
3. Define your local environment properties at `/opt/myst/lib/default.properties`
4. Define a `deploy.properties` under a given source directory to indicate the deployment metadata
5. Ensure your artifact is built.

After this, you can deploy from any directory with the `deploy.properties` by simply running `myst-deploy.sh`

## Install the MyST CLI

Install the MyST CLI to `/opt/myst` using instructions available [here](https://myst.rubiconred.com/webhelp/myst-cli-user-guide-5.6.0.0/index.html#install_myst.html).

## Create the wrapper script

Create a `myst-deploy.sh` script and make it available on the `PATH`

```
export MYST_HOME=/opt/myst
PATH=$PATH:$MYST_HOME
DATETIME=`date +%s`
export MYST_WORKSPACE=/tmp/$DATETIME
mkdir -p $MYST_WORKSPACE/conf
cp deploy.properties $MYST_WORKSPACE/conf
cp -r target $MYST_WORKSPACE/target
myst deploy-new -artifacts [x] deploy
rm -rf $MYST_WORKSPACE
```

## Define your local environment properties

This can be achieved by creating a file at `/opt/myst/lib/default.properties` with all of the required local environment properties. Below is an example:

```
core.domain.name=soa_domain
core.fmw.admin.password=OMITTED
core.fmw.version=12.2.1.3.0
core.node[machine1].authentication.credentials.password=OMITTED
core.node[machine1].authentication.credentials.username=oracle
core.node[machine1].host=fmw1
core.node[machine1].product-id=wls-admin,bpm
core.product[rcu].param[db-password]=OMITTED
core.product[rcu].param[db-sys-password]=OMITTED
core.product[rcu].param[db-url]=jdbc:oracle:thin:@database:1521:orcl
core.product[rcu].param[db-user-prefix]=DEV
```

The passwords above will be encrypted on the first run. If you require any deploy-time customisation values (i.e. stream model properties), these can be added into the `default.properties` file as well.

## Create the deploy metadata for your source code

Create a `deploy.properties` in the same directory as the `pom.xml`

Below is an example for an ADF application:
```
core.deployment[x].artifact.location=target/StockQuote_UK.war
core.deployment[x].param[deployment-order]=550
core.deployment[x].param[deployment-plan]=(EMBEDDED)adf/META-INF/plan.xml
core.deployment[x].param[myst-config-plan-apply]=true
core.deployment[x].param[myst-config-plan-location]=(EMBEDDED)/adf/META-INF/myst-config-plan.xml
core.deployment[x].param[redeploy]=true
core.deployment[x].param[name]=StockQuote_UK
core.deployment[x].param[target]=${core.domain.cluster[soa].name}
core.deployment[x].type=application
```

## Run local deployment

Ensure the application is built and exists at the location indicated by the `core.deployment[x].artifact.location` property.

Run `./myst-deploy.sh` to deploy the built-artifact to your local environment.