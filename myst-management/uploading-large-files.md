Need a way to upload files that are too large for the Myst Support Portal? You can use our **S3** storage.

## Getting the Credentials
1. Log into https://mystsoftware.freshdesk.com/
2. On the homepage you will find **S3 Credentials** containing your ID and secret key.
<br>![](/myst-management/img/uploading-large-files-01.png)

## Using AWS CLI

#### Setup
Download and install [AWS CLI](https://aws.amazon.com/cli/) or use a AWS S3 UI client.

#### Uploading Files
Replace `access key`, `secret key` and `bucket name` with your information.

```
# Variables - replace these with yours
ACCESS_KEY=123
SECRET_KEY=456
CUSTOMER=ABC

# Setup profile in $HOME/.aws/credentials
aws configure set profile.mystsupport.aws_access_key_id $ACCESS_KEY
aws configure set profile.mystsupport.aws_secret_access_key $SECRET_KEY
aws configure set profile.mystsupport.output json
aws configure set profile.mystsupport.region us-west-2

# Upload the file:
aws --profile mystsupport s3 cp $MYST_BACKUP s3://myst-support-bau/customer/$CUSTOMER/
```
