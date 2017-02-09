## {{page.title}} 

If you follow the official [Oracle Fusion Middleware Developing Applications Using Continuous Integration](https://docs.oracle.com/middleware/1212/core/MAVEN/introduction.htm) documentation you may hit some known issues when it comes to building SOA, OSB and ADF components. The majority of these issues are caused by defects in the parent seed process.

We recommend that you raise a ticket with [Oracle Support](http://support.oracle.com) if you are stuck so that in the case of a defect, the root cause can be addressed in the next Oracle Middleware version. In addition to this we have provided below a reference of workarounds to the standard Oracle Maven build parent seed process in case it helps to unblock you.

### 12.1.3 Oracle Maven Synchornization and Workarounds

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

### 12.2.1 Oracle Maven Synchornization and Workarounds

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


