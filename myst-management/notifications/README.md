# MyST Notifications

MyST allows for the execution of custom actions in the event of a failure or success of an action. This can be a useful way to establish notification to other services such as Email and Slack.

## Example: Slack Notifications from MyST

Below is a working example of how to establish slack notifications from MyST.
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

### Sample Code

Be sure to replace the hardcoded channel below with the actual identifier.

#### MyST Info Notification to Slack

```
import os
def myst(cfg):
 action_name=cfg.configuration.getRequest().getAction()
 message = action_name.capitalize() + " success on " + cfg['internal.admin.url']
 os.system("curl -X POST -H 'Authorization: Bearer "+ cfg['slack.token.password'] +"' -H 'Content-type: application/json' --data '{\"channel\":\"G8F3FN40J\",\"text\":\""+message+"\",\"username\":\"MyST slack bot\"}' https://slack.com/api/chat.postMessage")
```
 
#### MyST Error Notification to Slack

```
import os
def myst(cfg):
 action_name=cfg.configuration.getRequest().getAction()
 message = "ERROR: " + action_name.capitalize() + " failed on " + cfg['internal.admin.url']
 os.system("curl -X POST -H 'Authorization: Bearer " + cfg['slack.token.password'] + "' -H 'Content-type: application/json' --data '{\"channel\":\"G8F3FN40J\",\"text\":\""+message+"\",\"username\":\"MyST slack bot\"}' https://slack.com/api/chat.postMessage")
```