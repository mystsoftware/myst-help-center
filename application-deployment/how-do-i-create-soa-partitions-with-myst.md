## {{page.title }} 

Yes.

You can create SOA partitions with MyST as indicated in the [MyST Supported Artifacts](https://docs.rubiconred.com/myst-studio/appendix/artifact/#sca). At deploy-time if you have a partition defined in the MyST metadata for any given SCA Artifact (aka Composite) and the partition does not exist, it will be auto-created. If the `composite.partition` is defined in the Maven `pom.xml` at SCA build-time and the MyST plugin was used to push the artifact to Studio, MyST will automatically discover the partition and use that to verify it exists at deploy-time and create it if it does not. For example, to set `Stock` as the partition name for a given SCA artifact, the following can be added within the `properties` of a Maven `pom.xml`.
```
<composite.partition>Stock</composite.partition>
```

Alternatively, you could use the MyST Studio  `Control` > `Custom` option under a given Platform Instance to create the partition directly before deployment using `create-soapartition` action and setting the name as a property (e.g. `-Dsca.composite.partition=Stock`).