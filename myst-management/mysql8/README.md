This article is about how to use Myst on MySQL8

# Assumptions
* MySQL8 images were introduced into Myst as part of the [7.2.0-rc1 release](https://www.mystsoftware.com/post/myst-studio-7-2-0-rc1). You'll need to download the mysql8 compatible image for the associated version.

# How to download the Myst Mysql8 image
There are two locations where you can download new releases. Either:
* On our website on the [Releases page](https://www.mystsoftware.com/resources/releases) under the specific release version, or
* Through our image repository

If using our image repository, from version 7.2.0-rc1 there will be two Myst Studio images created with each release.

1. The standard version - this uses the mysql 5 connector. For example, `7.2.0-rc1`.
1. An updated MySQL8 version - this uses the suffix `-mysql8` onto the version. For example, to get the first available MySQL8 version, the release number is `7.2.0-rc1-mysql8`.


# Docker compose file
Below is an example docker-compose.yml file. The key sections are:
* The MySQL image uses `mysql:8.0` from the official mysql repository at [https://hub.docker.com/_/mysql](https://hub.docker.com/_/mysql)
* There is a volume for the `/var` directory, where the database's data is stored.
    * This assures the data doesn't need to be exported and imported as part of a migration (if coming from MySQL 5+), or future upgrades


```
version: '2'
services:
  web:
    image: myst-studio
    container_name: myststudio_web
    links:
     - db
    volumes:
     - ./data/license:/usr/local/tomcat/conf/fusioncloud/license
     - ./data/ext:/usr/local/tomcat/conf/fusioncloud/ext
    restart: unless-stopped 
  db:
    image: mysql:8.0
    container_name: myststudio_db
    environment:
     - MYSQL_ROOT_PASSWORD=welcome1
     - MYSQL_DATABASE=fusioncloud 
     - MYSQL_USER=fusioncloud
     - MYSQL_PASSWORD=welcome1
    # Comment the lines below unless you want to expose the database to the outside world
    #ports:
    # - "3306:3306"
    volumes:
     - /var/lib/mysql
    restart: unless-stopped 
  https:
    image: nginx
    container_name: myststudio_https
    ports:
     - "5555:80"
     - "443:443"
    links:
     - web
    volumes:
     - ./data/nginx/nginx.conf:/etc/nginx/nginx.conf
     - ./data/nginx/ssl:/etc/nginx/ssl
    restart: unless-stopped 
```

# Running a fresh Myst install using MySQL8
There is an additional parameter called ``log_bin_trust_function_creators`` that needs to be added to MySQL8 if running with a fresh installation - i.e. not an existing database/data file. This is shown in the docker-compose.yml snippet below.

```
  ...
  db:
    image: mysql:8.0
    container_name: myststudio_db
    command: --log_bin_trust_function_creators=1
    environment:
   ...
```

