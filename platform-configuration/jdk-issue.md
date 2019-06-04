
### Symptom

During execution of MyST, it fails with a message similar to the following:

```
Determined Java version as '1.7.0_79'
Java version '1.7.0_79' lower than required: FMW version '12.2.1.0.0' requires at least JDK 8
```

### Solution

You need to ensure JDK 8 is installed on the target hosts.

If the correct JDK is already installed and you are still getting this issue, then you should make sure that the System Java is not still pointing to the old version.

If you cannot change the System Java, then you can override the `java` used by adding the following to `~/.bashrc` for the SSH user that is used by MyST:

```
JAVA_HOME=/usr/local/java/bin
PATH=$JAVA_HOME/bin:$PATH
```

Where `/usr/local/java/bin` is the location of the Java `bin` directory on your target host.