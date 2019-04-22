## Packet Size Breach / Support Artifact failure

### Diagnosis:

Packet size *(max_allowed_packet_size)* can be altered in the 'db' docker service, for issues faced during deployments. Validation can be done once the failed deployments are successfully deployed, after increasing the packet size.

One or more of the following issues are observed:

- Myst Studio hanging issues

- MyST Studio pushes users out while deploying

- Users unable to login into MyST Studio

  The following errors may appear in the logs:

- SqlExceptionHelper - Packet for query is too large
- Due to bigger in size, not able to persist support artifact
- logExceptions Packet for query is too large

![](C:\Users\admin\Desktop\000\sql.jpg)

### Cause:

MyST Deployment-activities sometimes may contain large number of Artifacts, which generate support Artifacts. These Support-Artifacts need to get uploaded to the Database, and the ***max_allowed_packet_size*** setting by-default is not big enough to allow them. Hence, the large number of failures in deployments, is due to the Support-Artifacts exceeding the limit of the default-value.

### Resolution:

 The resolution to push the large sized support artifacts to the 'db', is by setting the ***max_allowed_packet_size*** to a larger value in the 'docker-compose.yml'. Setting the value to 100M should be large enough to cover most cases. This allows the artifacts to pass to the 'db'. After increasing the value, please make sure to restart the service, for the changes to take effect.

Please find the process to change the ***max_allowed_packet size***, below:

1. Add the line below to increase the max_allowed_packet to 32 megabytes to the 'db' docker-compose.yml, in docker service. Please be mindful of the formatting (spaces).
   ***command: --max_allowed_packet=32M***

   ![](C:\Users\admin\Desktop\000\088.jpg)

2. Stop using *stop.sh*

3. Start using *start.sh*  -  this will recreate the database container with the new max_allowed_packet changes.



*The value/size of the **max_allowed_packet** can be set (to any value), based on one's requirement*