Read the instructions below if you have installed Artifactory with Myst and want to setup HTTPS (SSL).

# Prerequisites

2. JKS keystore (example here uses the keystore filename `identity.jks`)
3. SSL server certificate in JKS keystore (example here uses the certificate alias `artifactory`)

# Update Myst docker-compose Files

### Expose the SSL Port in docker-compose.yml

Port `8443` is exposed to you as SSL. Use this port to access Artifactory. Don't forget the 's' on http**s**. For example:
https://artifactory.mystsoftware.com:8443

1. Update the `/opt/myst-studio/conf/maven/docker-compose.yml` to include '`- 8084:8443`'. Indentation is important.
```yml
version: '2'
services:
  repo:
    image: maven-repository
    container_name: maven-repository
    ports:
     - 8083:8081
     - 8084:8443
    environment:
     - START_TMO=600
    restart: unless-stopped
  web:
    links:
     - repo
  ci:
    links:
     - repo
```

2. 


# Enable Artifactory SSL

### Copy Keystore to Artifactory and Enable SSL in Tomcat

1. Copy your keystore into the Artifactory Docker container.
```shell
docker exec -ti maven-repository /bin/sh -c 'mkdir -p /opt/jfrog/artifactory/data/keystore'
docker cp identity.jks maven-repository:/opt/jfrog/artifactory/data/keystore/
```

2. Update Tomcat `server.xml` to enable the SSL port.
```shell
# Backup server.xml
docker exec -ti maven-repository /bin/sh -c 'cd /opt/jfrog/artifactory/tomcat/conf/server.xml && cp -p server.xml server.xml.org'
docker cp identity.jks maven-repository:/opt/jfrog/artifactory/data/keystore/

# Replace server.xml with the contents below which includes SSL
docker exec -ti maven-repository /bin/sh -c 'cat > /opt/jfrog/artifactory/tomcat/conf/server.xml <<EOF
<Server port="8015" shutdown="SHUTDOWN">

    <Service name="Catalina">
        <Connector port="8081"/>

        <!-- This is the optional AJP connector -->
        <Connector port="8019" protocol="AJP/1.3"/>

        <!-- Define a SSL Coyote HTTP/1.1 Connector on port 8443 -->
        <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                   maxThreads="200"
                   scheme="https"
                   secure="true"
                   SSLEnabled="true"
                   keystoreFile="/opt/jfrog/artifactory/data/keystore/identity.jks"
                   keystorePass="changeit"
                   clientAuth="false"
                   sslProtocol="TLS"/>

        <Engine name="Catalina" defaultHost="localhost">
            <Host name="localhost" appBase="webapps"/>
        </Engine>
    </Service>
</Server>

EOF
'
```

# Add Trusted Certificates to Clients

The server's public certificate needs to be added to clients' truststores for a successful SSL handshake. In the Myst ecosystem truststores need to be updated on:
* Jenkins -> Artifactory
	* build jobs uploading artifacts to Artifactory
* Maven -> Artifactory
	* `mvn` commands executed by Myst to download artifacts for FMW deployment

### Jenkins Truststore

If the Jenkins SSL Truststore has not yet been setup as part of [Enable SSL (HTTPS) for Jenkins](myst-management/jenkins-ssl/README.md) then follow the instructions there.

### Maven Truststore

There are many options to add the arguments to Maven. [Maven documentation](https://maven.apache.org/guides/mini/guide-repository-ssl.html)  has information. Here are three options.

Add a flag to the Maven `settings.xml` so Maven is aware of the truststore to use.

##### Option 1 - Java cacerts

Go to your $JAVA_HOME/jre/lib/security/cacerts and specify the 

##### Option 2 - .mavenrc

On each AdminServer 
`$HOME/.mavenrc`
```shell
MAVEN_OPTS="$MAVEN_OPTS -Djavax.net.ssl.trustStore=/location/of/truststore/truststore.jks"
```

##### Option 3 - Environment Variable

Similar to Option 2, you can define an environment variable for the SSH user in `.bashrc` or `.bash_profile`.
```shell
export MAVEN_OPTS="$MAVEN_OPTS -Djavax.net.ssl.trustStore=/location/of/truststore/truststore.jks"
```

# (Optional) Disable HTTP - non-SSL

To disable Artifactory's HTTP (non-SSL) comment out the port HTTP port.

    ports:
     #- 8083:8081
     - 8084:8443

Restart the docker containers via Myst
```shell
cd /opt/myst-studio/bin
./stop.sh
./start.sh
```

