### Installing the License
For new installations of MyST Studio follow the instructions in the MyST Installation Guide:
https://docs.rubiconred.com/myst-studio/installation/myst-studio/#installation-using-the-wizard

### Updating the License
#### MyST Studio
##### Prerequisites
1. Run the `backup-database.sh` script to backup your MyST Studio database. This will allow you to easily rollback from an upgrade, if needed.
2. Ensure you have a license bundle from Rubicon Red.

#####5.6.5+
1. Log into MyST Studio
1. Administration > Renew License<br>![](img/myst-license-renew-license.png)
1. Select the license file (tar.gz) provided by Rubicon Red.
1. Log off then log on
1. In the bottom left are your new license details

**NOTE:** If you do not see the Renew License link press CTRL+F5 on your browser to refresh the cache.

#####PRE-5.6.5
It is recommended to upgrade to the latest version of MyST Studio to apply the license. If you have questions please contact [MyST Support](mailto:myst.support@rubiconred.com "myst.support@rubiconred.com").

#### MyST CLI
1. Follow the [MyST CLI license instructions](https://myst.rubiconred.com/webhelp/myst-cli-user-guide-5.6.0.0/index.html#install_myst_update_license.html) for either an automatic or manual installation

**NOTE:** If you have followed the instructions above please be aware that MyST will continue to display the old license until it expires. Once expired MyST will automatically apply the new license.

### What about the MyST.lic file I use to receive?
When renewing your license the `MyST-<name>-.lic.tar.gz` file now comes bundled with a license (`MyST.lic`) and an API key (`api.key`).

**NOTE:** If your version of MyST only requires `MyST.lic` you can simply extract the license file from `MyST-<name>.lic.tar.gz`.

### License Expired or Expiring?
If your license is due to expire and you are yet to receive the license please contact [MyST Support](mailto:myst.support@rubiconred.com "myst.support@rubiconred.com")
