## {{page.title}}

In 12c, Oracle introduced out-of-the-box support for Maven builds of SOA, OSB and ADF. As there was no direct support in 11g, Rubicon Red provided this capability through a series of supported parent pom.xml files. 

For MyST specific artifact types such as SQL and Application Configurations, you do not need to make any changes to your pom.xml when you migrate from 11g to 12c.

To upgrade an artifact from 11g to 12c, it simply has to be re-opened in JDeveloper 12c. Then, the change needs to be re-committed to source control and the artifact should be re-built. 

Before we can re-build out artifact for 12c, we need to:
 1. [seed the Oracle Maven parent POMs to our local or remote Maven repository](maven-11g-to-12c.md)
 2. update our project pom.xml files (as per below).
 
Below are some examples of 11g pom.xml files and the corresponding equivalents for 12c.

### SOA Suite POM Examples

#### 11g Project Object Model

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <groupId>com.rubiconred.myst.myst-ci</groupId>
    <artifactId>sca-build-11.1.1.7</artifactId>
    <version>2.2</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>StockTraderOrder</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0-SNAPSHOT</version>  
  <properties>
    <oracle.soa.home>/u01/app/oracle/product/fmw/Oracle_SOA1</oracle.soa.home>
  </properties>
</project>
```

#### 12c Project Object Model

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>StockTraderOrder</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>sar</packaging>
  <parent>
    <groupId>com.oracle.soa</groupId>
    <artifactId>sar-common</artifactId>
    <version>12.2.1-0-0</version>
  </parent>
  <properties>
    <scac.input.dir>${project.basedir}/</scac.input.dir>
    <scac.input>${scac.input.dir}/composite.xml</scac.input>
    <oracleHome>/u01/app/oracle/product/fmw1221/Oracle_SOA1</oracleHome>
  </properties>
  <build>
    <plugins>
        <plugin>
            <groupId>com.oracle.soa.plugin</groupId>
            <artifactId>oracle-soa-plugin</artifactId>
            <version>12.2.1-0-0</version>
            <configuration>
                <composite>${scac.input}</composite>
                <excludes>
                    <exclude>**/pom.xml</exclude>
                    <exclude>**/SCA-INF/</exclude>
                </excludes>
            </configuration>
            <extensions>true</extensions>
        </plugin>
    </plugins>
  </build>
</project>
```

### Oracle Service Bus POM Examples

#### 11g Project Object Model

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <groupId>com.rubiconred.myst.myst-ci</groupId>
    <artifactId>osb-build-11.1.1.7</artifactId>
    <version>2.0</version>
    <relativePath></relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>stock-services</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
</project>
```

#### 12c Project Object Model

```
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd" xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.oracle.servicebus</groupId>
        <artifactId>sbar-project-common</artifactId>
        <version>12.2.1-0-0</version>
    </parent>
    <groupId>com.acme</groupId>
    <artifactId>stock-services</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>sbar</packaging>
</project>
```