If you are behind a forward proxy server for internet access and you need to perform tasks that accessing internet \(such as installing MyST Studio\), you will need to setup the forward proxy server settings in different applications as this post shown:

Note: In this post, sometime the escape character '%5C' is used in place of backslash '\'. Some applications will function with backslash, but some don't and must use backslash escape character.

# Forward Proxy Configurations {#jive_content_id_Forward_Proxy_Configuations}

## Shell {#jive_content_id_1_Shell}

This step is required for general command prompt internet access via the forward proxy.

Note: This level of configuration is for the current shell only, you can permanently apply this configuration in ~/.bashrc or ~/.bash\_profile.

### Steps

```
export http_proxy='http://<Domain>%5C<Username>:<Password>@<ProxyHost>:<ProxyPort>'

export https_proxy='<Protocol>://<Domain>%5C<Username>:<Password>@<ProxyHost>:<ProxyPort>'

export  no_proxy="127.0.0.1, localhost,<CiServerIPHostName>,<CiServerIP>"
```

### Testing HTTP

```
wget[http://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm!\[\]\(/assets/shell\_test\_http.png](http://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm![]%28/assets/shell_test_http.png)\)
```

### Testing HTTPS

```
wget[https://github.com/docker/compose/releases/download/1.14.0/docker-compose-$\(uname-s\)-$\(name](https://github.com/docker/compose/releases/download/1.14.0/docker-compose-$%28uname-s%29-$%28name) -m\)
```

![](/assets/shell_test_https.png)

## Yum {#jive_content_id_2_Yum}

This step is required for yum to access internet resources via the forward proxy.

Note: This level of configuration is for yum only.

### Steps

```
vi /etc/yum.conf

proxy=[http://](https://rubiconred.jiveon.com/external-link.jspa?url=http%3A%2F%2F)<ProxyHost>:<ProxyPort>  
proxy\_username=<Username>  
proxy\_password=<Password>
```

### Testing

If you can install xclock, run the following command to test yum internet connectivity.

```
sudo yum install xclock
```

## Docker {#jive_content_id_3Docker}

This step is required for docker to access internet resources via the forward proxy.

Note: This level of configuration is for docker service only, it doesn't automatically apply proxy configuration to the docker container \(e.g. /bin/bash\).

### Steps

```
cat <<EOF \| sudo tee -a /etc/sysconfig/docker  
http\_proxy='[http://](https://rubiconred.jiveon.com/external-link.jspa?url=http%3A%2F%2F)<Domain>%5C<Username>:<Password>@<ProxyHost>:<ProxyPort>'  
https\_proxy='<Protocol>://<Domain>%5C<Username>:<Password>@<ProxyHost>:<ProxyPort>'  
EOF
```

Note: The no\_proxy configuration is not required here, because docker service use these proxy setting to pull images from internet, and docker won't be communicating to the CI server at all \(or any internal servers\). However, there should also be no harm of setting no\_proxy here.

```
sudo sed -i '/\\[Service\\]/a EnvironmentFile=/etc/sysconfig/docker' /usr/lib/systemd/system/docker.service  
sudo systemctl daemon-reload  
sudo service docker restart
```

### Testing

Run command 'docker info' and check the Http Proxy and Https Proxy settings where '\' backslash is replaced with '%5C'.![](/assets/docker_test_docker_info.png)

Run any command that contain docker related internet activity e.g. /opt/app/myst-studio/bin/start.sh, if run successfully, then the proxy is configured correctly.

## Jenkins \(For NTLM Forward Proxy\) {#jive_content_id_4Jenkins_For_NTLM_Forward_Proxy}

**This step is required for jenkins to access internet resources via the NTLM authentication forward proxy server.**

This step is required for Jenkins to access internet resources via the forward proxy.

Note: This level of configuration is for Jenkins only, this proxy setting doesn't automatically apply on any other applications.

### Steps \(For NTML Forward Proxy Only\)

Jenkins cannot communicate to proxy server that requires NTML Authentication.The solution is to force the JVMto enable "basic" authentication \(Not NTML Authentication\).

For information regarding NTML Authentication Issue:

-[java - Jenkins proxy 407 error - Stack Overflow](https://rubiconred.jiveon.com/external-link.jspa?url=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F29682844%2Fjenkins-proxy-407-error)

* [\[JENKINS-3350\] Connect to update center via HTTP proxy that requires NTLM authentication - Jenkins JIRA](https://rubiconred.jiveon.com/external-link.jspa?url=https%3A%2F%2Fissues.jenkins-ci.org%2Fbrowse%2FJENKINS-3350)

The solution to this problem is to modify the jenkins yml file:

```
vi $MYSTSTUDIO\_HOME/conf/ci/docker-compose-base.yml
```

Add the following line after the Port definition:

```
environment:
* JAVA\_OPTS=-Dhttp.auth.preference="basic"
```

The Jenkins JVM will use basic authentication to the proxy, not NTLM authenication.

Once the yml file is modified, run /myst-studio/bin/restart.sh to bounce the jenkins container \(This will also bounce all other docker containers\).

### Steps \(For Any Forward Proxy\)

Provide the proxy configuration in the Jenkins console. For example:![](/assets/jenkins_step_console_config.png)

Note: The Advanced > Validate Proxy button will return "Failed to connect to[http://www.google.com](https://rubiconred.jiveon.com/external-link.jspa?url=http%3A%2F%2Fwww.google.com)\(code 407\).", but the connection should still work. Simply ignore this error.

### Testing

Jenkins Console > Jenkins > Manage Jenkins > Manage Plugins > Click Check Now and make sure there is no error.

## Artifactory \(For NTLM Forward Proxy\) {#jive_content_id_5_ArtifactoryFor_NTLM_Forward_Proxy}

**This step is required for Artifactory to access internet resources via the NTLM authentication forward proxy server.**

This issue is similar to the Jenkins issue \(4\).

Note: This level of configuration is for Artifactory only, this proxy setting doesn't automatically apply on any other applications.

### Steps \(For NTML Forward Proxy Only\)

Artifactory cannot communicate to proxy server that requires NTML Authentication. The solution is to force the JVM to enable "basic" authentication \(Not NTML Authentication\).

The solution to this problem is to modify the jenkins yml file:

```
vi $MYSTSTUDIO\_HOME/conf/maven/docker-compose.yml
```

Add the following line after the Port definition:

```
environment:
    JAVA\_OPTS=-Dhttp.auth.preference="basic"
```

The Artifactory JVM will use basic authentication to the proxy, not NTLM authenication.

Once the yml file is modified, run /myst-studio/bin/restart.sh to bounce the artifactory container \(This will also bounce all other docker containers\).

### Steps \(For Any Forward Proxy\)

Provide the proxy configuration in the Artifactory console. For example:

![](/assets/artifactory_step_proxy_config.png)

### Testing

Artifactory Console > Admin > Remote > Click on any Remote Repository \(e.g. jcenter\) > Click Test and make sure there is no error.

## Maven {#jive_content_id_6_Maven}

**This step is required for maven to access internet resources. Note: This step isn't required if Artifactory is configured with Maven in settings.xml, because Artifactory will access Internet on behalf of Maven.  **

### Steps

Add the below section to the global maven settings.xml file right after the root <settings> element.

```
<proxies>  
<proxy>  
<id>proxy</id>  
<active>true</active>  
<protocol>http</protocol>  
<username><Domain>%5C<Username></username>  
<password><Password></password>  
<host><ProxyHost></host>  
<port><ProxyPort></port>  
<nonProxyHosts><NoProxy></nonProxyHosts>  
</proxy>  
</proxies>
```

### Testing Prerequisite

Follow the below steps:

```
mkdir /tmp/TestMaven

vi pom.xml
```

Add the below content and save the file.

```
<project>  
<modelVersion>4.0.0</modelVersion>  
<groupId>com.mycompany.app</groupId>  
<artifactId>my-app</artifactId>  
<version>1</version>  
</project>
```

### Testing For Maven

Run run the following command to test Maven's Internet connectivity.

```
mvn -U -X clean install
```

### Testing For Maven + Artifactory \(If Artifactory is configured in Maven settings.xml\)

Perform Step \(5\) to configure Artifactory to have correct Internet connectivity.

Run run the following command to test Maven's Internet connectivity via Artifactory.

```
mvn -U -X clean install
```



