Myst docker based [installation](https://userguide.mystsoftware.com/6.6.0/installation/myst-studio/) comes with a docker compose file which creates a mysql docker container myststudio_db if we want to run myst studio on custom database like Amazon Aurora database

#### Update Myst database details

- cd /opt/myst-studio/conf
- Update the environment details in below docker-compose.yml

**docker-compose.yml**

```
version: '2'
services:
  web:
    image: myst-studio
    container_name: myststudio_web
    links:
     - db
    # Comment the lines below if you want to just allow only https access. i.e. Enabling HTTPS
    # ports:
    # - "8085:8080"
    volumes:
     - ./data/license:/usr/local/tomcat/conf/fusioncloud/license
     - ./data/ext:/usr/local/tomcat/conf/fusioncloud/ext
    environment:
     - MYST_DB_PASSWORD=welcome1
     - MYST_DB_NAME=fusioncloud
     - MYST_DB_USERNAME=fusioncloud
     - MYST_DB_HOST=db
     - MYST_DB_PORT=3306
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

- Replace the docker-compose.yml
- cd /opt/myst-studio/bin
-  ./start.sh