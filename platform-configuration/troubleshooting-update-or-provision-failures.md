When running a MyST action such as **update** or **provision** you may encounter errors. Here you can find ways to help troubleshoot the errors.

## MyST update (or create-resource) failure
If a MyST update action fails you can look at logs to have a rough idea of where and how the failure occurred.
1. `wlst.log` shows the WLST commands run with the last entry being the failure.
2. `myst-diagnostic.log` provides debugging and a `dumpStack()` of WLST upon error

## MyST create-domain failure
The create-domain action is a subset of the MyST Provision or Re-Provision action. If create-domain fails there are log files help identify the root cause.

1. `wlst.log` shows the WLST commands run with the last entry being the failure.
2. `myst-diagnostic.log` provides debugging and a `dumpStack()` of WLST upon error
3. `<myst_workspace>/wlstOfflineLogs_appsoa/wlst_YYYYMMDD.log` provides an extremely valuable capture of create-domain information.
   For example if you specify the same ports twice this log file will show a port clash when creating a domain.

## Null Pointer Exception (NPE)
The NPE generally means a required MyST property has not been set OR has been overriden to be blank (null). For blank (null) see [During execution I get a java.lang.NullPointerException](/platform-configuration/javalangnullpointerexception/javalangnullpointerexception.md) for more troubleshooting information.

If a resource was recently updated/added to MyST it's possible there is a missing property. For example when creating a JDBC DataSource the NAME property may have been missed.

## MyST update fails but changes are activated in WebLogic
When MyST runs an update a number of events occur. MyST uses Weblogic's session to make changes. It's been identified in particular cases the session activates however, MyST reports an attempted rollback (which doesn't actually occur).

Here is a high level explanation:

1. MyST update action is executed to update the username of a WebLogic DataSource
2. MyST opens a WLST session and performs updates - `edit()` and `startEdit()`
3. MyST activates the session successfully - `activate()`
4. WLST throws an error after successfully activating. This means changes have been applied in WebLogic.
5. The WLST error is likely to be the WebLogic attempting to validate connectivity with the JDBC DataSource
6. MyST catches the error and reports to the user a failure occurred. 

#### Workaround
Unfortunately due to how WebLogic throws the exception **after** activating changes we can perform the following workaround.

Run a MyST **check-for-drift** or **dryrun** to validate (and correct) any unexpected changes that have taken effect.

*In future versions of MyST we may investigate a way to handle these particular scenarios.*
