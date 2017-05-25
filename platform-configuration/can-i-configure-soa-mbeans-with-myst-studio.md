Yes, SOA Mbeans can be configured using MyST Studio.

Below is a working example for configuring some of the commonly tuned SOA Mbean properties.

# Step 1: Edit the Platform Blueprint
1. Navigate to the Platform Blueprint which you want to change in MyST Studio ("Modeling" > "Platform Blueprints")
2. Click "Edit Configuration"
3. Go to "Global Variables". 
4. Click "Edit" then "Edit Bulk".
5. Add the following properties.
```
oracle.sysman.emas.discovery.wls.FMW_DISCOVERY_MAX_CACHE_AGE=2592000000
oracle.sysman.emas.discovery.wls.FMW_DISCOVERY_MAX_WAIT_TIME=1800000
oracle.sysman.emas.discovery.wls.FMW_DISCOVERY_USE_CACHED_RESULTS=true
LargeRepository=true
```
6. Click "Done" then "Save".
7. Go to "Products" > "Oracle SOA Suite". 
8. Click "Edit" then under "Name-Value Parameters" click "Edit Bulk".
9. Add the following properties. (Note: audit-level should already exist so only needs to be edited - not created)
```
audit-level=Production
bpel-AuditLevel=production
bpel-BpelcClasspath=/u03/app/oracle/admin/domains/tbpm5_domain/mserver/tbpm5_domain/lib/gen_credentials_client-1/0.jar:/u03/app/oracle/product/12.1.3/tbpm5_domain/soa/soa/modules/oracle.bpm.composer.webapp_11.1.1/commons-codec-1.2.jar
bpel-RecoveryConfig.RecurringScheduleConfig.maxMessageRaiseSize=50
bpel-RecoveryConfig.RecurringScheduleConfig.startWindowTime=00:00
bpel-RecoveryConfig.RecurringScheduleConfig.stopWindowTime=23:59
bpel-RecoveryConfig.RecurringScheduleConfig.subsequentTriggerDelay=900
bpel-RecoveryConfig.StartupScheduleConfig.maxMessageRaiseSize=0
bpel-RecoveryConfig.StartupScheduleConfig.startupRecoveryDuration=0
bpel-RecoveryConfig.StartupScheduleConfig.subsequentTriggerDelay=900
bpel-SyncMaxWaitTime=200
bpmn-RecoveryConfig.RecurringScheduleConfig.maxMessageRaiseSize=0
bpmn-RecoveryConfig.RecurringScheduleConfig.startWindowTime=00:00
bpmn-RecoveryConfig.RecurringScheduleConfig.stopWindowTime=00:00
bpmn-RecoveryConfig.RecurringScheduleConfig.subsequentTriggerDelay=900
bpmn-RecoveryConfig.StartupScheduleConfig.maxMessageRaiseSize=0
bpmn-RecoveryConfig.StartupScheduleConfig.startupRecoveryDuration=0
bpmn-RecoveryConfig.StartupScheduleConfig.subsequentTriggerDelay=900
soa-infra-EMDurationWindowUnits=minutes
soa-infra-EMDurationWindowVal=1
```
10. Click "Done" then "Save".
11. Go to "WebLogic Domains" and expand the domain.
12. Then, go to "WebLogic Clusters" and click on the SOA cluster.
13. Set the "Frontend Host" to tbpm5.testservices.rabobank.nl and "Frontend Http Port" to 80.
14. Click "Save".
15. Click "Apply Changes"

# Step 2: Update Platform Model based on the Blueprint changes
1. Go "Modeling" > "Platform Models"
2. Select the Platform Model corresponding to the previously edited Platform Blueprint
3. Go to "Actions" and select "Update"
In the dialog box select "Update to latest draft" (assuming the Platform Blueprint was edited in the same version that the model is currently on).
4. Click "Update".

# Step 3: Run Configure SOA Action
Now that the Platform Model has been updated to incorporate the recent Platform Blueprint changes, we can run the Configure SOA action.
1. From the Platform Model go to to "Actions" and select "Control"
2. Select the "Custom" action and enter the "Command" as configure-soa.

3. Click "Execute"
4. Once the action has completed click "View" under "Execute Log" to see the changes made.

## Configure SOA advanced properties as part of Provisioning
The above SOA configuration step can be configured to automatically run at the end of platform provisioning by adding action.configure.post=configure-soa in the Global  Variables of the Platform Blueprint

# Summary of Advanced SOA Properties
This is a non-exhaustive list of available SOA/BPEL/EM/Mediator/BPMN Mbeans attributes that can be configured in MyST Studio.

| **EM Attribute** | **Component in MyST Studio** | **Name-Value Parameter in MyST Studio** | **Available Since** |
| -- | -- | -- | -- |
| SoaInfraConfig &gt; AuditLevel | Blueprint &gt; Products &gt; | audit-level |  2.0.0+ |
| SoaInfraConfig &gt; EMDurationWindowVal | Blueprint &gt; Products &gt; Oracle SOA Suite | soa-infra-EMDurationWindowUnits                                         4.0.0+ |
| SoaInfraConfig &gt; EMDurationWindowVal | Blueprint &gt; Products &gt; Oracle SOA Suite | soa-infra-EMDurationWindowVal                                           4.0.0+
| SoaInfraConfig &gt; ServerURL | Blueprint &gt; WebLogic Domains &gt; (domain name) &gt; WebLogic Clusters &gt; (soa cluster) | Automatically calculated from "Frontend Host" and "Frontend Http Port" | 2.0.0+ |
| SoaInfraConfig &gt; CallbackServerURL | As above | As above | 2.0.0+ |
| MediatorConfig &gt; AuditLevel | Blueprint &gt; Products &gt; Oracle SOA Suite | audit-level | 1.0.0+ |
| BPELConfig &gt; AuditLevel | Blueprint &gt; Products &gt; Oracle SOA Suite | audit-level | 1.0.0+ | 
| BPELConfig &gt; SyncMaxWaitTime | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-SyncMaxWaitTime | 4.0.0+ |
| BPELConfig &gt; BpelcClasspath | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-BpelcClasspath | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; maxMessageRaiseSize | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.RecurringScheduleConfig.maxMessageRaiseSize | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; startWindowTime | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.RecurringScheduleConfig.startWindowTime | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; stopWindowTime |  Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.RecurringScheduleConfig.stopWindowTime | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; subsequentTriggerDelay | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.RecurringScheduleConfig.subsequentTriggerDelay | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; maxMessageRaiseSize | Blueprint &gt; Products &gt; Oracle SOA Suite |  bpel-RecoveryConfig.StartupScheduleConfig.maxMessageRaiseSize | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; startupRecoveryDuration | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.StartupScheduleConfig.startupRecoveryDuration | 4.0.0+ |
| BPELConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; subsequentTriggerDelay | Blueprint &gt; Products &gt; Oracle SOA Suite | bpel-RecoveryConfig.StartupScheduleConfig.subsequentTriggerDelay | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; maxMessageRaiseSize | Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.RecurringScheduleConfig.maxMessageRaiseSize | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; startWindowTime | Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.RecurringScheduleConfig.startWindowTime | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; stopWindowTime | Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.RecurringScheduleConfig.stopWindowTime | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; RecurringScheduleConfig &gt; subsequentTriggerDelay | Blueprint &gt; Products &gt; Oracle SOA Suite |  bpmn-RecoveryConfig.RecurringScheduleConfig.subsequentTriggerDelay | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; maxMessageRaiseSize| Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.StartupScheduleConfig.maxMessageRaiseSize | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; startupRecoveryDuration | Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.StartupScheduleConfig.startupRecoveryDuration | 4.0.0+ |
| BPMNConfig &gt; RecoveryConfig &gt; StartupScheduleConfig &gt; subsequentTriggerDelay | Blueprint &gt; Products &gt; Oracle SOA Suite | bpmn-RecoveryConfig.StartupScheduleConfig.subsequentTriggerDelay | 4.0.0+ |
| em.props &gt; oracle.sysman.emas.discovery.wls.FMW\_DISCOVERY\_USE\_CACHED\_RESULTS  | Global Variables | oracle.sysman.emas.discovery.wls.FMW_DISCOVERY_USE_CACHED_RESULTS | 2.5.1+ |
| em.props &gt; oracle.sysman.emas.discovery.wls.FMW\_DISCOVERY\_MAX\_CACHE\_AGE | Global Variables | oracle.sysman.emas.discovery.wls.FMW\_DISCOVERY\_MAX\_CACHE\_AGE | 2.5.1+ |
| em.props &gt; oracle.sysman.emas.discovery.wls.FMW\_DISCOVERY\_MAX\_WAIT\_TIME | Global Variables |                                                          oracle.sysman.emas.discovery.wls.FMW\_DISCOVERY\_MAX\_WAIT\_TIME | 2.5.1+ |
|  em.props &gt; LargeRepository  | Global Variables | LargeRepository | 4.0.0+ |


