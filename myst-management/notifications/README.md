# MyST Notifications

MyST allows for the execution of custom actions in the event of a failure or success of an action. This can be a useful way to establish notification to other services such as Email and Slack.

## Example: Slack Notifications from MyST

Below is a working example of how to establish slack notifications from MyST.
Before working through these steps, please make sure you are on version 5.8.6 or later.

1. Create an API token in Slack. We will use this later to connect from MyST.
2. From MyST Studio, create two new custom actions.
   - slack-info-notification 
     - **Target Location**: ext/targets/jython/slack-error-notification.py
     - **Resource Type**: Jython Action
     - **Description**: Notify Slack - ERROR Message
   - slack-error-notification 
     - **Target Location**: ext/targets/jython/slack-error-notification.py
     - **Resource Type**: Jython Action
     - **Description**: Notify Slack - ERROR Message
3. ...