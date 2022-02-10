Need a way to upload files that are too large for support.rubiconred.com? You can use our **S3** storage.

1. Get the **Credentials**
2. Use **AWS CLI** or a **UI client**<br><br>

## Getting the Credentials
1. Log into https://rubiconred.freshdesk.com/
2. On the Homepage you will find an S3 Credentials page
<br>![](/myst-management/img/uploading-large-files-01.png)
<br>**NOTE**: If this page does not exist contact **[MyST Support](mailto:myst.support@rubiconred.com)** or mention this in your **support ticket**.<br><br>

## Using the AWS CLI

#### Setup
Download and install [AWS CLI](https://aws.amazon.com/cli/)

#### Uploading Files
Replace `access key`, `secret key` and `bucket name` with your information.

```
# Setup profile
aws configure set profile.mystsupport.aws_access_key_id <Access Key ID>
aws configure set profile.mystsupport.aws_secret_access_key <Secret Access Key>
aws configure set profile.mystsupport.output json
aws configure set profile.mystsupport.region us-west-2

# Upload the file, for example:
aws --profile mystsupport s3 cp myst-studio-6.5.0-1530228410.sql s3://myst-support-bau/customer/home-made-cookies/
```
* Creates a profile named mystsupport
* Profile will contain the access key and the secret key along with the output format and region
* The last command uploads the file.<br><br>
