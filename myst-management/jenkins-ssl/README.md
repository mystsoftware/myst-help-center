This guide details the steps required to enable HTTPS on the Jenkins docker container. It follows Jenkins documentation: [https://www.jenkins.io/doc/book/installing/initial-settings/#configuring-http](https://www.jenkins.io/doc/book/installing/initial-settings/#configuring-http)

# Prerequisites
## Keystores
It is assumed you have SSL certificates provided.
* `identity.jks` is your identity store with the server certificate
* `trust.jks` is your trust store with any trusted certificates (eg. organization CA certs)

# Setting Up HTTPS
## Creating Keystores Directory
Create a new directory under the Myst server installation and copy your keystore there named `identity.jks` and `trust.jks`.
```bash
mkdir /opt/myst-studio/conf/data/keystores
```

## Adding Myst Certificate to trust.jks
Jenkins will publish application metadata to Myst. This is done during the build process. If untrusted, you will see errors during the build process when Jenkins publishes application metadata to Myst.

There are two options. **Use one option only.**
1. Reuse existing truststore or;
2. Create a new truststore

##### Option 1: Reuse existing Truststore
1. Copy the existing Jenkins Java `cacerts`
```bash
cd /opt/myst-studio/conf/data/keystores
docker cp myststudio_ci:/etc/ssl/certs/java/cacerts trust.jks
```

##### Option 2: Create a New Truststore
1. Go to the keystores directory
```bash
cd /opt/myst-studio/conf/data/keystores
```

2. Retreive the certificate from Myst. Change `localhost:443` to your myst `hostname:port`.
```bash
echo -n | openssl s_client -connect localhost:443 | openssl x509 > myst.crt
```

3. Load the myst certificate into the truststore.
```bash
keytool -importcert -noprompt -trustcacerts -alias myst -file myst.crt -keystore trust.jks -storepass changeit
```

## Updating docker-compose-base.yml
1. Make a backup of the `docker-compose-base.yml`
2. Update  `/opt/myst-studio/conf/ci/docker-compose-base.yml`.
``` yaml
version: '2'
services:
  ci:
    image: ci-server
    container_name: myststudio_ci
    
    # Ports exposed from Jenkins (external_host:container)
    ports:
     - 8081:8080
     - 8443:8443
     - 50000:50000
    # Volumes - (external_host:container)
    volumes:
     - ../data/keystores:/var/jenkins_home/keystores
    
    # Environment Variables
    environment:
      JENKINS_OPTS: "--httpPort=-1 --httpsPort=8443 -Djavax.net.ssl.trustStore=/var/jenkins_home/keystores/trust.jks --httpsKeyStore=/var/jenkins_home/keystores/identity.jks --httpsKeyStorePassword=changeit"
    
    restart: unless-stopped
```

A detailed explanation of the changes are in the **Appendix**.

## Restart
Restarting will also restart Myst Studio, Artifactory, and Jenkins.
```shell
cd /opt/myst-studio/bin
./stop.sh
./start.sh
```

**And you're finished!**


# Appendix
## Explanation of YAML Changes
## Ports
1. Expose the HTTPS port such as `- 8443:8443`
2. Comment out `- 8081:8080` so HTTP is disabled

##### Options
* Change `8443` to anything you'd like such as `- 8443:443`
* To also allow **HTTP** uncomment `- 8081:8080`

```yaml
    # Ports exposed from Jenkins (external_host:container)
    ports:
     #- 8081:8080
     - 443:8443
     - 50000:50000
```

## Volumes
Links the keystores directory created earlier to the Jenkins container.

```yaml
    # Volumes - (external_host:container)
    volumes:
     - ../data/keystores:/var/jenkins_home/keystores
```

## Environment Variables
Environment variable used by Jenkins on startup.

##### Options
* The `--httpPort=-1` disables HTTP. Change to `--httpPort=8080`  if you want to allow HTTP too.

```yaml
    # Environment Variables
    environment:
      JENKINS_OPTS: "--httpPort=-1 --httpsPort=8443 -Djavax.net.ssl.trustStore=/var/jenkins_home/keystores/trust.jks --httpsKeyStore=/var/jenkins_home/keystores/identity.jks --httpsKeyStorePassword=changeit"
```

# Troubleshooting
Check the Jenkins docker logs on the Myst server:
`docker logs -f myststudio_ci`

# Issues
Raise a support ticket at [https://mystsoftware.freshdesk.com](https://mystsoftware.freshdesk.com) .
