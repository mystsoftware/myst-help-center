### Are you using 11g or 12?

This guide provide tips for using and troubleshooting the official 12c Maven Parent POMs and the Rubicon Red Maven Parent POMs provided for 11g.

### 12c Maven Build Tips 

If you follow the official [Oracle Fusion Middleware Developing Applications Using Continuous Integration](https://docs.oracle.com/middleware/1212/core/MAVEN/introduction.htm) documentation you may hit some known issues when it comes to building SOA, OSB and ADF components. The majority of these issues are caused by defects in the parent seed process.

We recommend that you raise a ticket with [Oracle Support](http://support.oracle.com) if you are stuck so that in the case of a defect, the root cause can be addressed in the next Oracle Middleware version. That said, we have provided below a reference of workarounds to the standard Oracle Maven build parent seed process in case it helps to unblock you.

#### 12.1.3 Oracle Maven Synchornization and Workarounds

Update the value of **ORACLE_HOME** to suit the target environment.

```
export ORACLE_HOME=/opt/app/oracle/product/fmw1221
export PATH=$PATH:${ORACLE_HOME}/oracle_common/modules/org.apache.maven_3.0.5/bin

# Workaround for building composites with OWSM policies as per https://community.oracle.com/thread/3597910
printf '<project xmlns="http://maven.apache.org/POM/4.0.0">\
  <modelVersion>4.0.0</modelVersion>\
  <groupId>com.oracle.adf</groupId>\
  <artifactId>wsm-policy-core</artifactId>\
  <version>12.1.3-0-0</version>\
  <packaging>jar</packaging>\
  <name>wsm-policy-core</name>\
  <description>Generated from Oracle JDeveloper</description>\
  <dependencies>\
    <dependency>\
      <groupId>com.oracle.adf</groupId>\
      <artifactId>osdt_xmlsec</artifactId>\
      <version>12.1.3-0-0</version>\
    </dependency>\
    <dependency>\
      <groupId>com.oracle.adf</groupId>\
      <artifactId>wsm-agent-core</artifactId>\
      <version>12.1.3-0-0</version>\
    </dependency>\
  </dependencies>\
</project>' > wsm-policy-core-12.1.3.pom
rm -f ${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/adf/wsm-policy-core/12.1.3/wsm-policy-core-12.1.3.pom
cp wsm-policy-core-12.1.3.pom ${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/adf/wsm-policy-core/12.1.3

# Dependency for workaround as per http://www.esentri.com/blog/2016/04/07/unable-to-compile-a-composite-java-embedded-maven/
mvn install:install-file \
-Dfile=${ORACLE_HOME}/soa/soa/modules/oracle.soa.bpel_11.1.1/orabpel.jar \
-DgroupId=com.oracle.soa -DartifactId=orabpel -Dversion=12.1.3.0-0 -Dpackaging=jar 

# Publishes the OFMW dependencies locally.
mvn install:install-file \
-Dfile=${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/maven/oracle-maven-sync/12.1.3/oracle-maven-sync-12.1.3.jar \
-DpomFile=${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/maven/oracle-maven-sync/12.1.3/oracle-maven-sync-12.1.3.pom
mvn com.oracle.maven:oracle-maven-sync:12.1.3-0-0:push \
-DoracleHome=${ORACLE_HOME} -DfailOnError=false

```

#### 12.2.1 Oracle Maven Synchornization and Workarounds

Update the value of **ORACLE_HOME** to suit the target environment.

```
ORACLE_HOME=/u01/app/oracle/product/fmw1221
PATH=$PATH:${ORACLE_HOME}/oracle_common/modules/org.apache.maven_3.2.5/bin

# Workaround documented at: https://rhpatrickblog.wordpress.com/2015/11/11/restoring-osb-12-2-1-maven-functionality/

rm -f ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/client/12.2.1/client-12.2.1.pom
cp client-12.2.1.pom ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/client/12.2.1

rm -f ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/sbar-system-common/12.2.1/sbar-system-common-12.2.1.pom
cp sbar-system-common-12.2.1.pom ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/sbar-system-common/12.2.1

rm -f ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/sbar-project-common/12.2.1/sbar-project-common-12.2.1.pom
cp sbar-project-common-12.2.1.pom ${ORACLE_HOME}/osb/plugins/maven/com/oracle/servicebus/sbar-project-common/12.2.1

# SOA build workaround
rm -f ${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/fmwshare/com.oracle.webservices.fmw.web-common-schemas-impl/12.2.1/com.oracle.webservices.fmw.web-common-schemas-impl-12.2.1.pom
cp com.oracle.webservices.fmw.web-common-schemas-impl-12.2.1.pom ${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/fmwshare/com.oracle.webservices.fmw.web-common-schemas-impl/12.2.1

# Dependency for workaround as per http://www.esentri.com/blog/2016/04/07/unable-to-compile-a-composite-java-embedded-maven/
mvn install:install-file -Dfile=${ORACLE_HOME}/soa/soa/modules/oracle.soa.bpel_11.1.1/orabpel.jar -DgroupId=com.oracle.soa -DartifactId=orabpel -Dversion=12.2.1.0-0 -Dpackaging=jar 

# Publishes the OFMW dependencies locally.
mvn install:install-file \
-Dfile=${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/maven/oracle-maven-sync/12.2.1/oracle-maven-sync-12.2.1.jar \
-DpomFile=${ORACLE_HOME}/oracle_common/plugins/maven/com/oracle/maven/oracle-maven-sync/12.2.1/oracle-maven-sync-12.2.1.pom
mvn com.oracle.maven:oracle-maven-sync:12.2.1-0-0:push \
-DoracleHome=${ORACLE_HOME} -DfailOnError=false

```

### 11g Maven Build Tips 

The MyST CLI ships with Maven Parent POMs for building 11g projects. After installing MyST CLI, you can seed the parent POMs as follows.

#### Installing the Maven artifacts for a local build

##### Installing the POM files

The below script can be used to install the parent poms into a local maven repository. 

```
# OSB
mvn clean install:install -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7-dependent.pom
mvn clean install:install -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7-standalone.pom
mvn clean install:install -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7.pom
  
# SOA
mvn clean install -f $MYST_HOME/lib/resources/maven/sca/adf-config/pom.xml
mvn clean install:install -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.x-dependent.pom
mvn clean install:install -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.1.1.6.pom
mvn clean install:install -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.1.1.7.pom
  
# MDS
mvn clean install:install -f $MYST_HOME/lib/resources/maven/mds/build-mds-project-11.1.1.7.pom
# JDeveloper
mvn clean install:install -f $MYST_HOME/lib/resources/maven/jdev/build-jdev-project-11.1.1.7.pom
```

##### Installing the library dependencies for OSB standalone build

Unlike the SOA and JDeveloper build, the OSB build can be achieved without the need to install OSB once the initial library dependency seed has been done from a machine with OSB installed. Here is how:

1. Run the script from a Linux machine which has OSB 11.1.1.7 installed.
2. Edit the file `install-dep-11.1.1.7.sh`, setting:
 * `MYST_FMW` to point to Oracle FMW home
 * `MYST_ARTIFACT_DEPLOY_PREFIX` to do a maven install rather than deploy. e.g (note that `MYST_ARTIFACT_DEPLOY_PREFIX` is commented out for deploy):
```
export MYST_FMW=/u01/app/oracle/product/fmw
export MYST_OSB=$MYST_FMW/Oracle_OSB1
export MYST_TEMP=/tmp
set -e
export MYST_ARTIFACT_DEPLOY_PREFIX=install:install-file
#export MYST_ARTIFACT_DEPLOY_PREFIX="deploy:deploy-file -Durl=http://admin:password@127.0.0.1:8083/artifactory/libs-release-local"
set -e
...
```

#### Publishing to a Maven Repository

This is a one-off activity that will allow all developers and the CI server to retrieve the build parents automatically on build. Before performing these steps, make sure you have the Maven `settings.xml` defined to point to your Maven Repository such as Artifactory, Nexus, Archiva etc.

Run the following commands, making sure that your replace the URL with your Maven repository URL (which contains login/pwd)

```
REPO_USERNAME=admin
REPO_PASSWORD=password
REPO_URL=127.0.0.1:8083/artifactory/libs-release-local
  
# OSB
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7-dependent.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7-standalone.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/osb/build-osb-11.1.1.7.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
  
# SOA
mvn deploy -f $MYST_HOME/lib/resources/maven/sca/adf-config/pom.xml -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.1.1.6.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.1.1.7.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/sca/build-sca-11.x-dependent.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
 
# MDS
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/mds/build-mds-project-11.1.1.7.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
 
# JDev
mvn deploy:deploy -f $MYST_HOME/lib/resources/maven/jdev/build-jdev-project-11.1.1.7.pom -DaltDeploymentRepository=orgrepo::default::http://$REPO_USERNAME:$REPO_PASSWORD@$REPO_URL
```

##### Deploying the library dependencies for OSB standalone build

1. Run the script from a Linux machine which has OSB 11.1.1.7 installed.
2. Edit the file `install-dep-11.1.1.7.sh`, setting:
* `MYST_FMW` to point to Oracle FMW home
* `MYST_ARTIFACT_DEPLOY_PREFIX` to do a maven deploy rather than install. e.g (note that `MYST_ARTIFACT_DEPLOY_PREFIX` is commented out for install):
```
export MYST_FMW=/u01/app/oracle/product/fmw
export MYST_OSB=$MYST_FMW/Oracle_OSB1
export MYST_TEMP=/tmp
set -e
#export MYST_ARTIFACT_DEPLOY_PREFIX=install:install-file
export MYST_ARTIFACT_DEPLOY_PREFIX="deploy:deploy-file -Durl=http://admin:password@127.0.0.1:8083/artifactory/libs-release-local"
set -e
...
```

