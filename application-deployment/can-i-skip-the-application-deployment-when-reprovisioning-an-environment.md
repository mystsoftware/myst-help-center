Yes, it is possible.

By default, after a reprovision, MyST will automatically deploy all of the previously deployed applications. But, if you wish to skip automatic re-deployment of all previously deployed applications after a platform reprovision you can clear the application cache.

The application cache can be cleared for all platform instances by executing the following:

1. Connect to the MyST repository via a MySQL client and run `use fusioncloud`
2. Clear the application cache for all instances with:
   `delete from platform_instance_artifact_property;`

For a pre-5.5 version of MyST run the following instead:

`delete from platform_instance_artifact;`

If you wish to clear the application cache for a specific platform instance you can run the following:

1. Connect to the MyST repository via a MySQL client and run `use fusioncloud`
2. Select the instance which you wish to clear the application cache on. This can be achieved by first finding the external\_id on the page url. For instance, if the Platform Instance page URL is https://acme-corp.myst.com/console/\#/platform-instances/f7f23a3d-1a54-420d-bde1-5eecbda8f3c5 then the external\_id is f7f23a3d-1a54-420d-bde1-5eecbda8f3c5 and we can use that to find the internal id with `select ID from PLATFORM_INSTANCE where external_id='f7f23a3d-1a54-420d-bde1-5eecbda8f3c5';`
3. Once we have the internal ID, we can remove all of the `PLATFORM_INSTANCE_ARTIFACT` records for that given platform instance. For instance if our platform\_instance\_id is 1, we would run `delete from PLATFORM_INSTANCE_ARTIFACT where platform_instance_id=1;`



