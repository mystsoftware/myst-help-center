Consider the following options to improving performance around MyST.

### Log file size / Packet Size exceeds

Log files/Logging service capture and provide all the necessary information from the instances. The file size of these log files eventually grow in size and could be the reason to cause few problems while they get pushed to the database. Similarly, the failed support artifacts are also pushed to the database and the length of the query could trigger the errors if breached.
Examples of errors are:
1. Packet for query is too large
2. Due to bigger in size, not able to persist support artifact to the database

Errors can be found in the myststudio_web container:
`ERROR o.h.e.j.s.SqlExceptionHelper   - Packet for query is too large (25134496 > 4194304)`

The log file errors are due to server logs not being able to be pushed to the database, as there is a limit in the size defined for the length of the query is larger than the maximum length. This can be solved by changing the value on the server by setting the ***max_allowed_packet*** variable.

Please find the process to change the ***max_allowed_packet size***, below:

1. Add the line below to increase the max_allowed_packet to 32 megabytes to the 'db' docker service. Please be mindful of the formatting (spaces).
    ***command: --max_allowed_packet=32M***
2. Stop using *stop.sh*
3. Start using *start.sh*  -  this will recreate the database container with the new max_allowed_packet changes.

*The value/size of the **max_allowed_packet** can be set (to anything), based on one's requirement.*

### Linux Entropy (random/urandom)

The OS may be running out of entropy. See the Oracle Support article below for more information. Changing from `/dev/random` to `/dev/./urandom` not only significantly improves OFMW but will improve MyST SSH connectivity time as MyST also relies on Java.
* [How to Diagnose a Linux Entropy Issue on WebLogic Server Instances](https://support.oracle.com/epmos/faces/DocumentDisplay?id=1574979.1)

### Platform Instance Action Checker
MyST Studio creates a background thread for every platform instance. When each thread executes it consumes seconds of CPU time to check for new actions against its platform instance.

As the the number of platform instances increase the threads will begin to overlap each other resulting in constant high CPU usage.

Use the `fc.quiet_period_sec.platform_instance` to reduce the platform instances' background check intervals. The default is `15` seconds.

The value can be roughly formulated to `<number_of_platform_instances> X 2`. For example, for 60 platform instances try experimenting with **120** (seconds).
`myststudio_web:/usr/local/tomcat/conf/fusioncloud/fc-configuration.properties`

```bash
fc.host.address=XXXXXXXXX
fc.quiet_period_sec.platform_instance=120
fc.port=XXXX
```

Don't forget to restart the container.
```bash
docker restart myststudio_web
```

### Raise a MyST Support Ticket
If the above fixes don't help with MyST performance issues then raise a [support ticket](https://support.rubiconred.com) for us to assist.
