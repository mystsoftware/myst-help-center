Certificate expired or setting up Myst for the first time?

# Assumptions
* you have already created a public and private certificate
* new certificates are in `/tmp/cert.crt` and `/tmp/cert.key` (otherwise change commands as required)
* myst installation home is in `/opt/myst-studio` (otherwise change commands as required)

# Update SSL Certificates in Apache
Myst uses Apache NGINX as its frontend. First we will update the certificate directly in NGINX.

```shell
# Backup
docker cp myststudio_https:/etc/nginx/ssl/cert.crt /tmp/cert.crt.`date +%Y%m%d-%H%M`
docker cp myststudio_https:/etc/nginx/ssl/cert.key /tmp/cert.key.`date +%Y%m%d-%H%M`

# Copy new certs
docker cp /tmp/cert.crt myststudio_https:/etc/nginx/ssl/
docker cp /tmp/cert.key myststudio_https:/etc/nginx/ssl/

# Restart NGINX docker container
docker restart myststudio_https
```

# Update SSL Certificates for Docker Compose
Myst uses Docker Compose for configuration and can recreate containers. Adding the certificate to the compose configuration means on recreation the certificates will be updated too.

```shell
# Copy new certificates to the Docker Compose configuration folders
cp -p /tmp/cert.crt /opt/myst-studio/conf/data/keystores
cp -p /tmp/cert.key /opt/myst-studio/conf/data/keystores
```

And you're done!

# Rollback
If for any reason you want to rollback to the older certificates you can use the backup files you created.

```shell
# Copy the backups into NGINX
docker cp /tmp/cert.crt.<date> myststudio_https:/etc/nginx/ssl/cert.crt
docker cp /tmp/cert.key.<date> myststudio_https:/etc/nginx/ssl/cert.crt

# Restart NGINX docker container
docker restart myststudio_https
```
