MyST allows each artifact build from a CI server to be auto-registered in MyST Studio at build-time. From MyST Studio, artifacts can be grouped into one of more Application Blueprints. However, if you wish to add a lot of artifacts to a blueprint then you may wish to do this via a programmable interface so that it can be performed in bulk. Bulk loading of artifacts and associating them with application blueprints can be facilitated with two automations:

1. Automated job creation to ensure all of the artifacts to be later included in an application blueprint are pre-seeded
2. Bulk automated association of artifacts within application blueprints using the REST APIs

### Automated job creation

CI build jobs can be [configured](https://docs.rubiconred.com/myst-studio/build/ci/) to automatically push to MyST Studio. This registration must be performed prior to any attempt to group Artifacts into Application Blueprints.

If you are using Jenkins for CI and need to create a large number of automated build jobs from scratch Groovy scripts are a great way to do this.

### Creation of application blueprints using REST APIs

Once artifacts are imported
- you can get a list of them
- each artifact ID can then be referenced into an ABP file

The following will give you the list of registered artifacts in MyST Studio:
```
curl -k -X GET -H "Authorization: Bearer $MYST_TOKEN" https://$MYST_HOST/api/v1/release-management/artifacts?topLevel=true&withBuilds=true&withVersions=true
```
