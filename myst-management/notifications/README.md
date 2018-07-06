# MyST Notifications

## The DevOps Feedback Loop

The "feedback loop" is a vital part of any DevOps toolchain. The "feedback loop" is how teams can be proactively notified when changes are happening to infrastructure or code and also when actions need to be taken such as fixing a deployment or test failure. MyST provides a feedback loop through it's sophisticated support for event notifications.

MyST supports two types of notifications
 - Default Notifications
 - Custom Notifications

## Default Notifications

MyST has a default email notification mechanism. When enabled, MyST users will receive notifications for significant events such as when they are required to approval or perform an Application Release.

### Configuring Email Notification via fc-configuration.properties

To configure via the fc-configuration.properties, update  `/usr/local/tomcat/conf/fusioncloud/fc-configuration.properties` with the following

```
smtp.user.name=<email>
smtp.password=<password>
smtp.host=smtp.gmail.com
smtp.port= 587
```

Once MyST is restarted (or the configuration is reload via the API), the password in the file will be automatically encrypted and the Email notification sender will be configured. If the configuration details are incorrect, you will see a related exception in the MyST log.

The following curl snippet shows us how to reload the configuration via the REST API.

```
curl -X POST -k https://${MYST_HOST}/api/v1/administration/configuration/reload \
  -H "Authorization: Bearer ${MYST_TOKEN}" \
  -H 'Content-Type: application/json'
  -d '{
    "reloadConfiguredProperties": true,
    "reloadArtifactMetadata": false,
    "reloadDictionaries": false
  }'
```  

### Configuring Email Notification via the REST API

MyST provides an API for establishing the one-time email notification configuration. To interact with the MyST REST API, an API key must first be obtained.

#### Retrieve API key from MyST Studio console

1. Login to MyST Studio with an administrator account
2. Click on "Administration" then select "Users"
3. Under the MySTAdministrator (API User) click on drop-down and select "Show API Key"
![](/myst-management/img/1.show-api-key.png)
4. Copy the key, we will use this later in our MyST workspace. If you want, you can generate a new key at any time.
![](/myst-management/img/2.api-key-view.png)

#### Get current configuration

Set `MYST_TOKEN` as an environment variable containing the MyST Studio API Key and `MYST_HOST` as an environment variable containing the MyST host.

Run the following
```
curl -k https://${MYST_HOST}/api/v1/administration/configuration/properties \
  -H "Authorization: Bearer ${MYST_TOKEN}" \
  -H 'Content-Type: application/json'
```

This should output something similar to the following:
```
{
  "data": {
    "fc.host.address": "localhost",
    "fc.quiet_period_sec.platform_instance": "15",
    "fc.quiet_period_sec.release_pipeline": "60",
    "fc.port": "8085",
    "fc.oauth.endpoint": "http://localhost:8080/security"
  },
  "meta": {
    "status": "SUCCESS",
    "code": "0",
    "timeStamp": "2018-07-06 03:44:15.019 +0000"
  }
}
```

#### Apply email configuration

```
curl -X PATCH -k https://${MYST_HOST}/api/v1/administration/configuration/properties \
  -H "Authorization: Bearer ${MYST_TOKEN}" \
  -H 'Content-Type: application/json'
  -d \
  '{
    "smtp.user.name": "<email>",
    "smtp.password": "<password>",
    "smtp.host": "smtp.gmail.com",
    "smtp.port": "587"
  }'
```

## Custom Notifications

MyST Custom Actions provide a simple but powerful event notification mechanism.

Below are some examples of custom notifications that can be achieved in MyST:
- When provisioning fails, I want to be notified
- When provisioning succeeds, I want to update a JIRA ticket
- When I am deploying, I want to tell everyone on Slack

Although MyST Custom Actions can be great for triggering notifications at various stages in a platform or application lifecycle, it should be noted that they can be really used for any action. Basically, a custom action allows for to say
> When MyST does X, I want it to also do Y...

When using a MyST Custom Action for the purpose of notifications, we write
a small function in Python, Java or Jython (i.e. Python which runs in the JVM and has access to Java libraries).

MyST allows for the execution of Custom Actions in the event of a failure or success of a Core MyST Action. This can be a useful way to establish notification to other services such as Email and Slack.

### Working Example: Slack Notifications from MyST

Below is a working example of establishing slack notifications from MyST.
Before working through these steps, please make sure you are on version 5.8.6 or later.

1. Create an API token in Slack. We will use this later to connect from MyST.
2. From MyST Studio, create two new custom actions.
   - slack-info-notification
     - **Target Location**: ext/targets/jython/slack-info-notification.py
     - **Resource Type**: Jython Action
     - **Description**: Notify Slack - ERROR Message
     - **Content**: See below sample code
   - slack-error-notification
     - **Target Location**: ext/targets/jython/slack-error-notification.py
     - **Resource Type**: Jython Action
     - **Description**: Notify Slack - ERROR Message
     - **Content**: See below sample code
3. Add the following **Global Variables** to any blueprint that you want to enable notifications on
   - **myst.action-on-failure**=slack-error-notification
   - **action.deploy.post**=slack-info-notification
   - **action.update.post**=slack-info-notification
   - **action.provision.post**=slack-info-notification
   - **slack.token.password**=OMITTED
4. Once the global variables are applied in any platform instance, notifications will be enabled for any action failure and for success of a deploy, update and provision action.

#### Sample Code

Be sure to replace the hardcoded channel below with the actual identifier.

##### MyST Info Notification to Slack

```
import os
def myst(cfg):
 action_name=cfg.configuration.getRequest().getAction()
 message = action_name.capitalize() + " success on " + cfg['internal.admin.url']
 os.system("curl -X POST -H 'Authorization: Bearer "+ cfg['slack.token.password'] +"' -H 'Content-type: application/json' --data '{\"channel\":\"G8F3FN40J\",\"text\":\""+message+"\",\"username\":\"MyST slack bot\"}' https://slack.com/api/chat.postMessage")
```

##### MyST Error Notification to Slack

```
import os
def myst(cfg):
 action_name=cfg.configuration.getRequest().getAction()
 message = "ERROR: " + action_name.capitalize() + " failed on " + cfg['internal.admin.url']
 os.system("curl -X POST -H 'Authorization: Bearer " + cfg['slack.token.password'] + "' -H 'Content-type: application/json' --data '{\"channel\":\"G8F3FN40J\",\"text\":\""+message+"\",\"username\":\"MyST slack bot\"}' https://slack.com/api/chat.postMessage")
```
