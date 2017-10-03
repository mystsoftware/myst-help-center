## Provisioning Oracle Data Integrator 12c
### Assumptions
#### Enterprise Install with J2EE Agent
Oracle Data Integrator 12.2.1.2.6 is an enterprise install with a J2EE agent.

#### ODI Agent Name
1. The ODI agent is named OracleDIAgent and doesn't need to be changed.
2. Changing the name of this agent requires ODI Studio > Creating a domain template containing the new agent name > unpacking the domain template

#### Manual Configuration of the Physical and Logical Agents
1. Instructions provided below for configuration
2. Requires either ODI Studio (GUI client) or ODI Console (web interface)

### Installation
#### Overview
1. Provision using MyST Studio
2. Create an Agent in the Master Repository using ODI Studio or ODI Console
3. Restart the ODI managed servers

### Create an Agent with ODI Console
#### ODI Console (Web Browser)
1. Access the ODI Console on `http://<HOST>:15101/odiconsole`
2. Select Master Repository and log in with an administrative user such as `SUPERVISOR`<br>![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-console-physicalagent-01.png)
3. Go to **Topology** > **Physical Architecture**
4. Right click on **Agents** and click **Create**<br>![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-console-physicalagent-02.png)<br><br>
5. Enter the following details to add the physical agent:
  `Name: OracleDIAgent`
  `Host: <HOST>`
  `Port: <PORT>`
  <br>**NOTE**: Use a load balanced URL for `<HOST>` if the environment is highly available
![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-console-physicalagent-03.png)

#### Validation
1. In the ODI Console right click and Test Connection the OracleDIAgent<br>![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-console-physicalagent-04.png)<br><br>
2. A successful connection message will display<br>![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-console-physicalagent-05.png)<br><br>
3. Go to the Enterprise Manager console to ensure the ODI target appears.<br>![](/platform-configuration/provisioning-oracle-data-integrator-12c/odi-validation-01.png)

### Create an Agent with ODI Studio
As an alternative to using the ODI Console you can follow Oracle's documentation to create an agent using ODI Studio:
https://docs.oracle.com/middleware/12212/lcm/ODING/GUID-3F74BFAB-99E9-4060-86E8-F70A8352A701.htm#GUID-6EEED355-F944-447F-A4CE-EA7BD9FE160C