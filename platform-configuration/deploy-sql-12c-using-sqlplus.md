In Oracle Fusion Middleware 12c the sqlplus client does not come packaged with the installation. MyST gives you the option of specifying the client. This article details the steps to deploy SQL with MyST using the SQL*Plus (sqlplus) client in 12c.

# Prerequisites
#### Linux Access
You will need `root` user access to install SQL*Plus rpm.

#### SQL*Plus Download
Download the client:
* As of writing the URL is: [Instant Client Downloads for Linux x86-64](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html) for 12.2.0.1.0
| Name | File |
| ------------- |:-------------:|
| Instant Client Package - Basic: All files required to run OCI, OCCI, and JDBC-OCI applications | instantclient-basic-linux.x64-12.2.0.1.0.zip (68,965,195 bytes) (cksum - 3923339140) |
| Instant Client Package - SQL*Plus: Additional libraries and executable for running SQL*Plus with Instant Client | instantclient-sqlplus-linux.x64-12.2.0.1.0.zip (904,309 bytes) (cksum - 2291973160) |

# Installation
1. Login as the `root` user
1. Install the client and additional libraries
```
yum install oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
yum install oracle-instantclient12.2-sqlplus-12.2.0.1.0-1.x86_64.rpm
```

# MyST Configuration
1. Add a global variable for the SQL*Plus product home:
![SQL Plus MyST Global Variable](/platform-configuration/deploy-sql-12c/deploy-sql-12c-01.png)<br>**NOTE:** The default directory that Oracle will install SQL*Plus is `/usr/lib/oracle/12.2/client64`
1. Run a MyST update action on the platform instance to the expected revision
1. Ensure your application metadata contains the sql client:
![SQL Plus MyST Global Variable](/platform-configuration/deploy-sql-12c/deploy-sql-12c-02.png)<br><br>
1. For MyST CLI this will be:
```
core.deployment[SQLArtifact].param[client]=sqlplus 
```
