Need a way to upload files that are too large for support.rubiconred.com? You can use our **S3** storage.

1. Get the **Credentials**
2. Use **AWS CLI** or a **UI client**<br><br>

## Getting the Credentials
1. Log into http://support.rubiconred.com
1. On the Homepage you will find an S3 Credentials page
<br>![](/myst-management/img/uploading-large-files-01.png)
<br>**NOTE**: If this page does not exist contact **[MyST Support](mailto:myst.support@rubiconred.com)** or mention this in your **support ticket**.<br><br>

## (OPTION 1) Using the AWS CLI

#### Setup
1. Download and install [AWS CLI](https://aws.amazon.com/cli/)

#### Uploading Files
The following commands will setup your profile for AWS CLI. Make sure to substitute the access key, secret key and bucket name where necessary.
```
# Setup profile
aws configure set profile.mystsupport.aws_access_key_id <Access Key ID>
aws configure set profile.mystsupport.aws_secret_access_key <Secret Access Key>
aws configure set profile.mystsupport.output json
aws configure set profile.mystsupport.region us-west-2

# Upload the file
aws --profile mystsupport s3 cp <file> <S3 External Bucket Name>

# For example:
aws --profile mystsupport s3 cp myst-studio-6.5.0-1530228410.sql s3://rxr/folder
```
* Creates a profile named mystsupport
* Profile will contain the access key and the secret key along with the output format and region
* The last command uploads the file.<br><br>

## (OPTION 2) Using a UI Client

#### Setup

1. Download a client.
1. In this example we use [CloudBerry Explorer](https://www.cloudberrylab.com/explorer/amazon-s3.aspx)
1. Alternatively other clients or command line tools are available

#### Uploading Files
1. Open the S3 Browser client
1. Click **File** > **New Amazon S3 Account** and fill in the details and click OK
<br>![](/myst-management/img/uploading-large-files-02.png)<br><br>
1. Select your new account from the dropdown
<br>![](/myst-management/img/uploading-large-files-03.png)<br><br>
1. Click the **External Bucket** icon
<br>![](/myst-management/img/uploading-large-files-04.png)<br><br>
1. Enter details of the external bucket
<br>![](/myst-management/img/uploading-large-files-05.png)<br><br>
1. Double click the external bucket
<br>![](/myst-management/img/uploading-large-files-06.png)<br><br>
1. You can now upload files to this bucket
1. Update the support ticket to let us know a file has been uploaded
