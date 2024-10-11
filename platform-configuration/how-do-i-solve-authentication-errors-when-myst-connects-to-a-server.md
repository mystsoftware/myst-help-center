# How do I solve authentication errors when MyST connects to a server?

This article addresses situations where you have configured a host in a pre-existing infrastructure provider using password authentication, and you receive the following error when either testing the connection or performing an action on that instance \(e.g. provision\)

```
Checking connectivity for the admin compute node - rxr.infra.Compute-1
For pre-existing infrastructure, unable to connect to the admin node effectively stops the whole workflow, hence aborting execution
Checking connectivity for the admin compute node - rxr.infra.Compute-1
For pre-existing infrastructure, unable to connect to the admin node effectively stops the whole workflow, hence aborting execution
java.lang.RuntimeException: Could not connect to the host - osb-05-tk and port - 22
        at com.rubiconred.fusioncloud.platform.core.provisioning.steps.SetNodeCredentialsAndCheckConnectivity.checkSSHConnection\(SetNodeCredentialsAndCheckConnectivity.java:191\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.steps.SetNodeCredentialsAndCheckConnectivity.checkSSHConnection\(SetNodeCredentialsAndCheckConnectivity.java:147\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.steps.SetNodeCredentialsAndCheckConnectivity.execute\(SetNodeCredentialsAndCheckConnectivity.java:65\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.steps.SetNodeCredentialsAndCheckConnectivity.execute\(SetNodeCredentialsAndCheckConnectivity.java:27\)
        at com.rubiconred.fusioncloud.common.workflow.StepBase.execute\(StepBase.java:17\)
        at org.apache.commons.chain.impl.ChainBase.execute\(ChainBase.java:191\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.PlatformInstanceOrchestratorImpl.orchestrate\(PlatformInstanceOrchestratorImpl.java:220\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.PlatformInstanceOrchestratorImpl$InstanceBackgroundJob.run\(PlatformInstanceOrchestratorImpl.java:176\)
Caused by: java.io.IOException: Password authentication failed.
        at com.trilead.ssh2.auth.AuthenticationManager.authenticatePassword\(AuthenticationManager.java:346\)
        at com.trilead.ssh2.Connection.authenticateWithPassword\(Connection.java:362\)
        at com.rubiconred.myst.remote.SSHConnection.doAuthentication\(SSHConnection.java:342\)
        at com.rubiconred.myst.remote.SSHConnection.connect\(SSHConnection.java:327\)
        at com.rubiconred.myst.remote.SSHConnection.executeCommand\(SSHConnection.java:93\)
        at com.rubiconred.fusioncloud.myst.runtime.RemoteConnection.executeCommand\(RemoteConnection.java:38\)
        at com.rubiconred.myst.remote.SSHConnection.executeCommand\(SSHConnection.java:67\)
        at com.rubiconred.fusioncloud.myst.runtime.RemoteConnection.executeCommand\(RemoteConnection.java:32\)
        at com.rubiconred.fusioncloud.platform.core.provisioning.steps.SetNodeCredentialsAndCheckConnectivity.checkSSHConnection\(SetNodeCredentialsAndCheckConnectivity.java:186\)
... 7 more

Caused by: java.io.IOException: Authentication method password not supported by the server at this stage.
        at com.trilead.ssh2.auth.AuthenticationManager.authenticatePassword\(AuthenticationManager.java:316\)
        ... 15 more
```

## General Items to Check

This type of error can occur for a number of reasons, but most commonly it is due to a misconfiguration of ssh on the target server. The first thing to check is the /etc/ssh/sshd\_config file, in particular the following parameter is set

**PasswordAuthentication yes**

## Issues when Running as a Different User

MyST Studio allows you to connect to a server as one user, and then perform the configuration tasks as another user. Behind the scenes, once logged into the shell MyST Studio will use the following command format to run commands as another user

```
sudo -i -u <SUDO_USER> bash -c "<COMMAND>"
```

Using the command above you can verify that the sudo is working correctly by logging to the target server, and manually executing the sudo command for the sudo user. Generally, it is recommended to provide the following access for the connection user to execute the sudo commands.

```
Cmnd_Alias      SHELLS=/usr/bin/sh,/usr/bin/ksh,/bin/sh,/bin/ksh
Cmnd_Alias      FULL=/usr/local/bin/*,/usr/bin/*,/bin/cp
User_Alias      MSYT_USERS=<user1-OS-ID>,<user2-OS-ID>
MYST_USERS      ALL=(oracle)!SHELLS,NOPASSWD:FULL
```



