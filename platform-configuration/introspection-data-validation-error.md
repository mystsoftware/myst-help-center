I am receiving "Data integrity constraints were violated while trying to persist the necessary entities" error when trying to introspect and environment

# The Problem {#jive_content_id_The_Problem}

When attempting to introspect a domain you receive the following error:

```
"errors" : [ {
    "code" : "MYST1012",
    "description" : "Data integrity violation",
    "message" : "Data integrity constraints were violated while trying to persist the necessary entities.",
    "severity" : "ERROR",
    "category" : "SYSTEM",
    "domain" : "MyST",
    "rootCause" : {
      "exceptionClass" : "org.springframework.dao.DataIntegrityViolationException",
      "message" : "could not execute statement; SQL [n/a]; constraint [null]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement",
      "stackTrace" : "org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint [null]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
     at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:255)
     at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:221)
     at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.translateExceptionIfPossible(AbstractEntityManagerFactoryBean.java:417)
     at org.springframework.dao.support.ChainedPersistenceExceptionTranslator.translateExceptionIfPossible(ChainedPersistenceExceptionTranslator.java:59)
     at org.springframework.dao.support.DataAccessUtils.translateIfNecessary(DataAccessUtils.java:213)
```

# The Cause {#jive_content_id_The_Cause}

For some reason there are duplicate properties defined within the JCA Adapter configuration. Often this is due to the parameters having slightly different case. e.g. privateKeyFile versus PrivateKeyFile. When MyST Studio goes to store these into the database it violates the db constraints

# The Solution {#jive_content_id_The_Solution}

Manually manipulate the Plan.xml for the JCA Adapter and remove the duplicate configuration. This document will step you through some of the tools to make this as easy as possible.

## Generate the Introspection JSON {#jive_content_id_Generate_the_Introspection_JSON}

By using the command line introspection utility we are able to generate the json representation of the domain that is pushed to MyST Studio. If you are not familiar with the command line utility, more information on it can be found [here](/platform-configuration/i-have-existing-environments-i-built-outside-of-myst-how-can-i-bring-them-into-the-control-of-myst.html)

### Getting the Utility {#jive_content_id_Getting_the_Utility}

The WebLogic Introspection agent can be obtained by running the following from the MyST Studio host.

`docker cp myststudio_web:/usr/local/tomcat/conf/fusioncloud/agent/<File Name> .`

where`<File Name>`is one of the following depending on the operating system where you want to run MyST Studio on.

| File Name | Operating System |
| :--- | :--- |
| weblogic-introspection-linux-x86\_64 | Linux 64 bit |
| weblogic-introspection-solaris-x86\_64 | Solaris 64 bit |
| weblogic-introspection-solaris-sparc | Solaris SPARC |

Once you have obtained the file from within the container, you can copy it to any host that you want to introspect from. To make it readily available as`weblogic-introspection`, you should copy it to a location on the PATH, rename it and ensure it has execute permissions. For example:

```
scp weblogic-introspection-linux-x86_64 oracle@acme-dev.as.cloud /tmp 
mkdir -p /home/oracle/bin 
cp /tmp/weblogic-introspection-linux-x86_64 /home/oracle/bin/weblogic-introspection 
export PATH=$PATH:/home/oracle/bin 
chmod +x /home/oracle/bin/weblogic-introspection
```

### Running the Utility {#jive_content_id_Running_the_Utility}

```
weblogic-introspection -skip-push -export <path to domain files>
```

this will generate a file called blueprint

## Reviewing the JSON {#jive_content_id_Reviewing_the_JSON}

There are many tools that you can use to view and edit json files. One that I particularly like is the "JSON Online Editor" which is an extension for Chrome. It easily formats the json, as well having support for searching and traversing.![](/assets/2017-06-20 11_47_48-JSON Editor Online - view, edit and format JSON online.png)

The size and scope of the json document make it very difficult to compare all of the parameters for each individual JCA Adapter configuration looking for duplicates using the Online JSON Editor. There is a tool called[jq](https://rubiconred.jiveon.com/external-link.jspa?url=https%3A%2F%2Fstedolan.github.io%2Fjq%2F) that proves great json manipulation, in a similar fashion to the unix command sed.

If you do not want to install jq, there is an [online version](https://rubiconred.jiveon.com/external-link.jspa?url=https%3A%2F%2Fjqplay.org%2F) \(jqplay\) of the editor, as shown in article.

1. Copy the contents of the blueprint file created by the introspection utility into the JSON field of jqplay
2. Past the following into the filter field. This will filter out all of the FtpAdapter entries, and generate a list of all of the parameters per adapter configuration. This can also be used for the JMS, and Database simply by changing 
   `rxr.wls.JcaAdapter-FtpAdapter` to `rxr.wls.JcaAdapter-JmaAdapter` and  `rxr.wls.JcaAdapter-DbAdapter` respectively. You can also traverse the document in the right hand panel without the filter to find any other adapters you wish to evaluate.

```
.initialVersion.serviceTemplate.nodeTemplates."rxr.wls.JcaAdapter-FtpAdapter".properties.instance[] |{name: .jndiName."_value",  params: [.param[].mystId."_value"] | sort} 
```

![](/assets/2017-06-20 13_13_55-jq play.png)

3. Either review the filtered list in qjplay or use your favourite text editor to look for duplicate entries. In the example below eis/sftp/JDEXeAppSvr has both `PrivateKeyFile` and `privateKeyFile`



