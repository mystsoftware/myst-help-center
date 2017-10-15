# Configuring EJB Transaction Timeout

MyST Studio supports the configuration of EJB transaction timeout which can be either achieved globally or at a domain level.

#### **Global EJB Transaction Timeout**

To configure EJB transaction globally

1. From MyST model editor, click **Edit Configuration**
2. Navigate through **Weblogic Domain** -&gt; **domain **-&gt; **JTA Transaction**
3. Once you click on **JTA Transaction**, click on **Generate**
   ![](/assets/generate.png)
4. Set the **Timeout Seconds** to the desired value
   ![](/assets/timeout.png)
5. Click on **Save** and **Commit **the changes
6. Run a **MyST update** action to reflect the changes to all beans in the environment

#### **SOA EJB Transaction Timeout**

To configure SOA EJB transaction timeout

1. From MyST model editor, click **Edit Configuration**
2. Navigate through **Products **-&gt; **Oracle SOA Suite**
3. Click on add one in the **Name-Value Parameters**
4. Add the key as **‘soa-ejb-timeout’** and set the value accordingly
   ![](/assets/SOA-EJB.png)
5. Click on **Save **and **Commit **the changes
6. Run a **MyST update** action to reflect the changes to all BPEL beans

_**Note**: At the moment, all the SOA -EJB beans are not supported, here is the list of supported SOA-EJB beans supported_

* BPELActivityManagerBean
* BPELAuditTrailBean
* BPELDeliveryBean
* BPELDispatcherBean
* BPELEngineBean
* BPELFinderBean
* BPELInstanceManagerBean
* BPELProcessManagerBean
* BPELSensorValuesBean
* BPELServerManagerBean
* CompositeMetaDataServiceBean

#### **BPM EJB Transaction Timeout**

To configure BPM EJB transaction timeout

1. From MyST model editor, click **Edit Configuration**
2. Navigate through **Products **-&gt; **Oracle Business Process Management**
3. Click on add one in the **Name-Value Parameters**
4. Add the key as **‘bpm-ejb-timeout’** and set the value accordingly
   ![](/assets/bpm.png)
5. Click on **Save **and **Commit **the changes
6. Run a **MyST update** action to reflect the changes to all BPM beans

_**Note**: At the moment, all the BPM -EJB beans are not supported, here is the list of supported BPM-EJB beans supported_

* BPMNDeliveryBean
* BPMNDispatcherBean
* BPMNEngineBean
* BPMNFinderBean
* BPMNSensorValuesBean
* BPMNServerManagerBean
* BPMNActivityManagerBean
* BPMNInstanceManagerBean
* PMNProcessManagerBean

#### **Reflect the Changes to your Environment**

1. Shutdown all of the SOA servers. This can be done using
**Note**: To reflect these changes to your environment the SOA servers must first be shutdown. This is to allow MyST to update the soa-infra application with the new configuration. Once you




