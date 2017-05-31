MyST has comprehensive support for introspecting existing WebLogic environments so that they can be managed within MyST Studio. When managed within MyST Studio, many automated operations can be performed on them including but not limited to:
 - Migration to the Cloud (Oracle or AWS)
 - Automated Upgrade (e.g. 11g to 12c)
 - Performing a configuration drift checks
 - Automated configuration updates/patching/app deployments
 - Start/stop/rolling restart operations 

If you are wanting to perform introspection using MyST Studio, it is recommended to be on version 5.6 and above. If you want to use MyST introspection on a pre-5.6 version, it is recommended to raise a ticket with [MyST Support](http://support.rubiconred.com) who can help you work through the process and specifically point you to the relevant documentation for your version of MyST.

## Introspecting a WebLogic Domain from MyST Studio

You can introspect a [Platform Blueprint ](https://docs.rubiconred.com/myst-studio/platform/blueprints/) and/or [Platform Model](https://docs.rubiconred.com/myst-studio/platform/models/) from a WebLogic domain home directly within MyST Studio to bring it under the control of MyST.
Details on how to do this are documented [here](https://docs.rubiconred.com/myst-studio/platform/introspection/).

It is easiest to introspect directly within MyST Studio rather than via the command-line because it provides a drop-down menu for selecting Compute Definition and the Host to introspect from.
At introspection time MyST Studio will directly connect to the host over SSH and discover the Blueprint details from the WebLogic domain.

## Introspecting a WebLogic Domain using the CLI

As an alternative to the introspection capability in MyST Studio, there is command-line (CLI) introspection agent that can be ran directly from the Admin Server host.
The CLI introspection agent will discover the configuration details from the WebLogic domain and push them directly to MyST Studio for on-going management.

### Prerequisites for using the WebLogic Introspection agent

#### Installing the WebLogic Introspection agent

The WebLogic Introspection agent can be obtained by running the following from the MyST Studio host.
`docker cp myststudio_web:/usr/local/tomcat/conf/fusioncloud/agent/<File Name> .`
where `<File Name>` is one of the following depending on the operating system where you want to run MyST Studio on.

| File Name | Operating System |
| ------ | ---------------- |
| weblogic-introspection-linux-x86_64 | Linux 64 bit |
| weblogic-introspection-solaris-x86_64 | Solaris 64 bit |
| weblogic-introspection-solaris-sparc | Solaris SPARC |

Once you have obtained the file from within the container, you can copy it to any host that you want to introspect from.
To make it readily available as `weblogic-introspection`, you should copy it to a location on the PATH, rename it and ensure it has execute permissions. For example:
```
scp weblogic-introspection-linux-x86_64 oracle@acme-dev.as.cloud /tmp
mkdir -p /home/oracle/bin
cp /tmp/weblogic-introspection-linux-x86_64 /home/oracle/bin/weblogic-introspection
export PATH=$PATH:/home/oracle/bin
chmod +x /home/oracle/bin/weblogic-introspection
```

#### Obtaining the MyST API Key

In order to push discovered data from the command-line introspection agent to Studio, you will need to retrieve and/or regenerate an API key from MyST Studio. This can be performed as follows:

1. Login to MyST Studio with an administrator account
2. Click on "Administration" then select "Users"
3. Under the MySTAdministrator (API User) click on drop-down and select "Show API Key"
3. Copy the key, we will use this later. If you want, you can generate a new key at any time.

#### Obtaining the IDs of referenced resources

When using the CLI Introspection Agent, at times, you may need to provide IDs to reference existing components within MyST Studio.
Common use cases for doing this are as follows:
- Customising the Platform Blueprint introspection to reference a specific [Compute Definition](https://docs.rubiconred.com/myst-studio/infrastructure/compute-definitions/) other than the default one that comes with MyST Studio.
- Customising the Platform Model introspection to reference a specific [Environment Type](https://docs.rubiconred.com/myst-studio/infrastructure/environment-types/) other than the default "CI" environment type that comes out-of-the-box with MyST Studio.
- Ensuring consistency between environments. In this case, you may have a Platform Blueprint and a Platform Model and now want to introspect a different Platform Model but reference the previous Platform Blueprint. For example, if you introspected the Blueprint for Production and now want to introspect the UAT Model while reusing the same Blueprint to ensure consistency of common (non-environment specific) settings with what is in production.
- Advanced customisation to the translation between the WebLogic Domain model and the abstract MyST model which allows the domain to be lifted and shifted.

The ID for a given resource can be obtained from the URL of the resource in MyST Studio. Below are detailed steps for obtaining the ID for given resources

##### Compute Definition references

[Compute Definitions](https://docs.rubiconred.com/myst-studio/infrastructure/compute-definitions/) are used to indicate operating system requirements for target hosts in MyST Studio. 

You can reuse an existing Compute Definition at introspection time, by referencing it's ID. This can be obtained as follows:
1. From the MyST Studio console navigate to **Infrastructure** > **Compute Definition**.
2. Click on **Edit** next to the **Compute Definition** that you want to use and take note of the ID in the URL.
The ID is the last part of the URL. For example, if the URL is `https://acme-corp.cloud/console/#/compute-definitions/508a8b36-dc15-4252-b95d-9619865866a9` then the ID is `508a8b36-dc15-4252-b95d-9619865866a9`

##### Environment Type references

[Environment Types](https://docs.rubiconred.com/myst-studio/infrastructure/environment-types/) are designed to help categorize Oracle Middleware Platform Instances for governance purposes. 

You can reuse an existing Environment Type at introspection time,  by referencing it's ID. This can be obtained as follows:
1. From the MyST Studio console navigate to **Infrastructure** > **Environment Types**.
2. Click on **Edit** next to the **Environment Types** that you want to use and take note of the ID in the URL.
The ID is the last part of the URL. For example, if the URL is `https://acme-corp.cloud/console/#/environment-types/2499e11e-7916-4b78-81a8-415cd9f34879` then the ID is `2499e11e-7916-4b78-81a8-415cd9f34879`

##### Infrastructure Provider references

[Infrastructure Providers](https://docs.rubiconred.com/myst-studio/infrastructure/) map to a data center or a cloud provider region and typically have a set of associated host, networks, credentials and other related infrastructure resources. When you reference an infrastructure provider at introspection time, MyST will check if the host and credentials exist and if not, it will add them. After introspection, the user can then go in and enter the values (key or password) for the SSH credentials so that environments running on the given hosts can be managed by MyST.

You can reuse an existing Infrastructure Provider at introspection time, by referencing it's ID. This can be obtained as follows:
1. From the MyST Studio console navigate to **Infrastructure** > **Infrastructure Providers**.
2. Click the **Infrastructure Providers** that you want to use and take note of the ID in the URL.
The ID is the last part of the URL. For example, if the URL is `https://acme-corp.cloud/console/#/infrastructure-providers/pre-existing/f0ae32b2-b49b-4b23-b23c-9493b2abbea0` then the ID is `f0ae32b2-b49b-4b23-b23c-9493b2abbea0`

##### Platform Blueprint references

If you want to introspect a Platform Model on top of an existing Platform Blueprint, you will need to provide the ID of the Platform Blueprint as part of the introspection. This ID can be obtained as follows:
1. From the MyST Studio console navigate to **Modeling** > **Platform Blueprint**.
2. Click on the given Platform Blueprint that you want to use for your model take note of the ID in the URL.
The ID is part of the URL. For example, if the URL is `https://acme-corp.cloud/console/#/platform-blueprints/c2a9a9c6-4dd6-44b4-8b7a-54a14a235802/1.0.0` then the ID is `c2a9a9c6-4dd6-44b4-8b7a-54a14a235802`

### Introspecting a Blueprint from a WebLogic Domain using the CLI

Once we have the Compute Definition ID that we want to use in our introspected blueprint, we can perform a Platform Blueprint introspection as follows:
`weblogic-introspection -host <host> -port 443 -key <api_key> -computeId <computeId> <domain_home>`
If you do not specific the `-computeId` flag, the agent will use the default Compute Definition that came with MyST Studio.

After introspection, login to MyST Studio and check that the Blueprint exists under **Modeling** > ** Platform Blueprints**.
Now that we have a Blueprint, we can create a new Platform Model based on that and re-provision an identical environment.

### Introspecting a Blueprint and Model from a WebLogic Domain using the CLI

As an alternative to introspecting a Platform Blueprint and creating our Platform Model on top of that manually, we can also introspect the Platform Blueprint and its corresponding Platform Model in one go.
This approach is especially useful when we want to bring an existing WebLogic Domain under the control of MyST rather than introspecting it to create a copy.

`weblogic-introspection -host <host> -port 443 -key <api_key> -model -infra -computeId <computeId> -envId <envId> -infraId <infraId> <domain_home>`

## Introspecting a Model and reusing an existing Blueprint

If we want to use an existing Platform Blueprint, we just need to provide the ID for the in addition to the other arguments (i.e. set `-bpId`)
`weblogic-introspection -host <host> -port 443 -key <api_key> -model -infra -bpId <blueprintId> -computeId <computeId> -envId <envId> -infraId <infraId> <domain_home>`

## Performing a Dry Run introspection

We can skip the actual push of an introspection and just test the process, by using the flag `-skip-push`

## Customising the introspection process

If you want to customise the introspection process, you can run the following to extract the XSLT translations to the file system
`weblogic-introspection -gen`
This will result in an output similar to the following:
```
Write to /home/oracle/.myst/introspection/config2myst.xslt
Write to /home/oracle/.myst/introspection/jdbc2myst.xslt
Write to /home/oracle/.myst/introspection/jms2myst.xslt
Write to /home/oracle/.myst/introspection/products.json
Write to /home/oracle/.myst/introspection/config2model.xslt
Write to /home/oracle/.myst/introspection/product-catalog.json
```

After this, you can edit any of the XSLT or JSON translators and they will be used as part of the introspection.

If you want to remove your customisations, you can run `-clean`. 
This will remove the local files, thus forcing the introspection to use the uncustomised translators on the next introspection.
`weblogic-introspection -clean`

When using introspection customisation from MyST Studio, 
you will need to make sure the customised XSLT and JSON files exist in `~/.myst/introspection` on any host you want to introspect from.

If you have made a customisation that you have found useful, consider submitting it to Rubicon Red engineering for inclusion in an upcoming release.