### Installing the License
For new installations of MyST Studio follow the instructions in the MyST Installation Guide:
https://docs.rubiconred.com/myst-studio/installation/myst-studio/#installation-using-the-wizard

### Updating the License
#### MyST Studio
######Prerequisite
1. No data is expected to be lost however, as a precaution run the `backup-database.sh` script to backup your MyST Studio database.
2. You have received a new license file from Rubicon Red.

######5.6.3+
Use the Studio UI to upload a new license.

1. Create a zip file with the MyST.lic and api.key files (no folders, just files in the root of the zip). For example MyST.zip containing MyST.lic and api.key.
1. Log into MyST Studio
1. Refresh browser cache CTRL+F5
1. Administration > Renew License<br>![](img/myst-license-renew-license.png)
1. Select the zip created in the step above.
1. Log off then log on
1. In the bottom left are your new license details

######5.6.2.0
Run a script from the 5.6.2.0 Studio installer to install the new license.

1. Copy the your license file `MyST.lic` to `/opt/myst-studio/conf/data/license` directory
1. Go to `/opt/myst-studio/bin`
1. Run `./install-license.sh`

######Pre-5.6.2.x
Run a command to copy the license file into the Studio docker container.

1. Copy the your license file `MyST.lic` to `/opt/myst-studio/conf/data/license` directory
1. Run the following commands to copy the license into the docker container
```
docker cp /opt/myst-studio/conf/data/license/MyST.lic myststudio_web:/usr/local/tomcat/conf/fusioncloud/license/
```
1. When the old license expires the new license will automatically install.

**NOTE1:** MyST will continue to display the old license. When expired the new license will automatically install.

**NOTE2:** As of 5.6.1.1 if the license expires the Admin user (at login) will be prompted to upload a new license. The uploaded file must be a zip file with the MyST.lic and api.key files (no folders, just files in the root of the zip). For example MyST.zip containing MyST.lic and api.key.

#### MyST CLI
1. Follow the [MyST CLI license instructions](https://myst.rubiconred.com/webhelp/myst-cli-user-guide-5.6.0.0/index.html#install_myst_update_license.html) for either an automatic or manual installation

### What about the MyST.lic file I use to receive?
When renewing your license the `MyST-<name>-.lic.tar.gz` file now comes bundled with a license (`MyST.lic`) and an API key (`api.key`).

**NOTE:** If your version of MyST only requires `MyST.lic` you can simply extract the license file from `MyST-<name>.lic.tar.gz`.

### License Expired or Expiring?
If your license is due to expire and you are yet to receive the license please contact [MyST Support](mailto:myst.support@rubiconred.com "myst.support@rubiconred.com")
