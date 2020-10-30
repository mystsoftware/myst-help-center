We can now manage the Myst plugin configuration for Jenkins. Useful for automating the provisioning of Jenkins instances.

### Prerequisites

1.   Jenkins version 2.222+ is installed
2.  myst studio jenkins plugin 6.6.0+ is installed



#### Updating the MyST studio Configuration

- Go to Manage Jenkins > Configure System >Myst Studio Configuration
- Update the Myst Studio configuration

![](img/JCaC-MyST-Studio-Configuration.png)

#### Jenkinsfile

Add below code in the Jenkins file to publish the artifact to MyST studio

```
mystPlugin doNotIntrospect: false , pomFileOverride: 'Stock/pom.xml', postBuildStatus: true
```

#####  Sample Jenkinsfile

```
#!groovy

node {
   def artifactoryUrl = "http://maven-repository:8081/artifactory/libs-release-local"
   def oracleHome = "/u01/app/oracle/product/fmw122130"
   def mvnHome = tool 'Maven'
   def mvnCommand = "${mvnHome}/bin/mvn "

   stage('Checkout Code') {
       checkout scm
   }
   stage('Increment Version') {
       sh mvnCommand + "-f Stock/pom.xml build-helper:parse-version versions:set '-DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}-${env.BUILD_NUMBER}'"
   }
   stage('Build Artifact') {
       sh mvnCommand + "-f Stock/pom.xml clean package -DoracleHome=${oracleHome} -DsoaOracleHome=${oracleHome} install:install org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy -DaltDeploymentRepository=central::default::${artifactoryUrl}"
   }
   stage('Publish Artifact') {
       mystPlugin doNotIntrospect: false , pomFileOverride: 'Stock/pom.xml', postBuildStatus: true
   }
    
```

