Consider the following options to improving performance around MyST.

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



## Tuning Memory parameters

Memory parameters can also be tuned to improve the performance. In order to tune the memory, you can apply the following parameters in the ***docker-compose.yml*** file, as shown below. Restarting Myst would make the changes take effect with the new values. 

```
    environment:
      CATALINA_OPTS: "-Xmx3072m -Xms3072m"
```

(Please be careful of the formatting)



## Improve Performance using Logging/Log rotation :

Performance can be improved by rotating log files, which can turn huge in size, resulting in MyST's performance degradation. We can apply log-rotation inside the ***docker-compose.yml*** file, as below..

Add the following under the required container, in the above file..
(Please be careful of the formatting)

```python
logging:
 options:
  max-size: "10k"
  max-file: "5"
```

![](C:\Users\admin\Desktop\000\logf.jpg)

You can set the values suitable as per your environment.

Restart MyST Studio for the changes to take effect.



### Raise a MyST Support Ticket

If the above fixes don't help with MyST performance issues then raise a [support ticket](https://support.rubiconred.com) for us to assist.
