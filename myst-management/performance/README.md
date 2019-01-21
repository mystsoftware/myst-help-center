When the MyST host CPU is high then UI response times may be affected due to resource constraints. Consider the following options to improving CPU performance.

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