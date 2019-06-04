The MyST REST APIs can be used to bulk load artifact property values for multiple environments based on existing property data. This can be useful for a case where properties and their values are pre-existing in other files or for a customer on an earlier version of MyST wanting to move properties outside of Application Models and into the Artifact Property Registry and Platform Instances.

The property keys will be registered by the MyST plugin at the final stage of automated build. MyST will automatically introspect the properties before pushing them to MyST Studio. This is necessary so that MyST can establish the relationship between the artifacts and the properties they require.

There is a way to map existing data to platform instances in MyST. In order to support this, we will need a mapping between our platform instance and our configuration file(s) which contain the property values for that given instance.

The typical flow is as follows:

1. Invoke the REST API for querying a list of platform models
2. Assuming one file per model, iterate over all the existing configuration files (containing property keys and values)
  1. Find out the platform model record from the list fetched based on the name of the model (or whatever other convention has been used)
  2. Determine the ID of the platform instance corresponding to the model
  3. Invoke the REST API for registering the artifact property values against the platform instance. The properties and values should be available as part of the data in the configuration file
  4. Optionally, invoke the REST API to commit the property values. This will trigger the execution of those active release pipelines whose artifacts get impacted by these property values
  5. If the commit API is not invoked, the properties would be saved in draft state only and not trigger any release pipeline executions. This gives the user a verification window where he / she can login to MyST and verify the saved artifact property values before committing them
  6. One can verify if the values got saved correctly by querying the artifact property values for the instance using the REST API for the same
3. In case we want to set the default values or descriptions for properties registered in the Artifact Property Registry, the REST API for the same can be used as documented below.

## Migrating from Application Models

The erstwhile approach of defining individual application models has now been removed and the new Stream Model approach (which just feeds off the artifact property values stored against platform instances) has taken its place.

If you have defined properties and values inside application models, a REST API has been provided to get a CSV dump of the existing models along with their property keys and values so that they can be migrated to the new Stream Model approach.

This dump can then be used as a similar starting point as documented above to load the properties and values in bulk against the platform instance.

## REST APIs

### Querying the list of platform models

`GET /api/v1/platform/models?dataFilters=instance`

Response Payload
```
{
  "data": [
    {
      "id": "09101da8-0406-4a2f-a105-16eea349957d",
      "name": "SOA CI",
      "resourceType": "PLATFORM_MODEL",
      "environmentType": {
        "id": "2499e11e-7916-4b78-81a8-415cd9f34879",
        "name": "CI",
        "description": "Continuous Integration Environment",
        "order": 1,
        "status": "ACTIVE",
        "allowDraft": false
      },
      "workspaces": [
        {
          "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
          "name": "Default",
          "description": "This is a default workspace.",
          "status": "ACTIVE"
        }
      ],
      "status": "ACTIVE",
      "currentInstance": {
        "id": "3d831aba-b467-4d4b-9bad-fdca04175caa",
        "name": "SOA, SOA CI, CI",
        "resourceType": "PLATFORM_INSTANCE",
        "environmentType": {
          "id": "2499e11e-7916-4b78-81a8-415cd9f34879",
          "name": "CI",
          "description": "Continuous Integration Environment",
          "order": 1,
          "status": "ACTIVE",
          "allowDraft": false
        },
        "workspaces": [
          {
            "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
            "name": "Default",
            "description": "This is a default workspace.",
            "status": "ACTIVE"
          }
        ],
        "status": "ACTIVE",
        "lifecycleState": "INITIAL",
        "dataBagRef": "d81445d7-d481-48d8-920c-0a76f0c8468a"
      }
    },
    {
      "id": "4d819b58-241e-43df-8e65-5fabb88c82ed",
      "name": "SOA TEST",
      "resourceType": "PLATFORM_MODEL",
      "environmentType": {
        "id": "cc96e088-7212-481b-a4a8-8f692722114c",
        "name": "TEST",
        "description": "Test Environment",
        "order": 2,
        "status": "ACTIVE",
        "allowDraft": false
      },
      "workspaces": [
        {
          "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
          "name": "Default",
          "description": "This is a default workspace.",
          "status": "ACTIVE"
        }
      ],
      "status": "ACTIVE",
      "currentInstance": {
        "id": "a6b134ac-b8b0-4d14-8351-ad86c87ebe00",
        "name": "SOA, SOA TEST, TEST",
        "resourceType": "PLATFORM_INSTANCE",
        "environmentType": {
          "id": "cc96e088-7212-481b-a4a8-8f692722114c",
          "name": "TEST",
          "description": "Test Environment",
          "order": 2,
          "status": "ACTIVE",
          "allowDraft": false
        },
        "workspaces": [
          {
            "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
            "name": "Default",
            "description": "This is a default workspace.",
            "status": "ACTIVE"
          }
        ],
        "status": "ACTIVE",
        "lifecycleState": "INITIAL",
        "dataBagRef": "b14d314b-551e-406e-b509-8319ad441146"
      }
    },
    {
      "id": "1ab06255-64a8-42c0-b39e-8afc6bb56d26",
      "name": "SOA PROD",
      "description": "---",
      "resourceType": "PLATFORM_MODEL",
      "environmentType": {
        "id": "e8cb8b24-e0b4-4733-9648-6ada48df1d67",
        "name": "PROD",
        "description": "Production Environment",
        "order": 3,
        "status": "ACTIVE",
        "allowDraft": false
      },
      "workspaces": [
        {
          "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
          "name": "Default",
          "description": "This is a default workspace.",
          "status": "ACTIVE"
        }
      ],
      "status": "ACTIVE",
      "currentInstance": {
        "id": "3a6c4a40-85ea-4db2-b9c7-4d0dc08f8f28",
        "name": "SOA, SOA PROD, PROD",
        "resourceType": "PLATFORM_INSTANCE",
        "environmentType": {
          "id": "e8cb8b24-e0b4-4733-9648-6ada48df1d67",
          "name": "PROD",
          "description": "Production Environment",
          "order": 3,
          "status": "ACTIVE",
          "allowDraft": false
        },
        "workspaces": [
          {
            "id": "6fafeb5a-0bcb-4683-8f57-e287ea7eebaf",
            "name": "Default",
            "description": "This is a default workspace.",
            "status": "ACTIVE"
          }
        ],
        "status": "ACTIVE",
        "lifecycleState": "INITIAL",
        "dataBagRef": "893960da-47d4-4b00-87c5-b93921337826"
      }
    }
  ],
  "meta": {
    "status": "SUCCESS",
    "code": "0",
    "timeStamp": "2017-05-10 22:26:21.495 +0530"
  }
}
```

This gives the list of models along with corresponding platform instances. The important parts of this data are the 'name' and 'currentInstance.id'.

### Saving the artifact property values for a platform instance

`PATCH /api/v1/platform/instances/{instance-id}/data-bag`

Request Payload
```
{
  "properties": [
    {
      "name": "key1",
      "value": "value1"
    },
    {
      "name": "key2",
      "value": "value2"
    }
  ]
}
```
 
### Committing the artifact property values for a platform instance

`POST /api/v1/platform/instances/{instance-id}/data-bag/commit`

Request Payload should be empty, i.e, `{}` 

### Retrieving the current saved artifact property values for a platform instance (may or may not be committed)

`GET /api/v1/platform/instances/{instance-id}/data-bag?filterBy=DRAFT`

Response Payload

```
{
  "data": [
    {
      "property": {
        "id": "1",
        "name": "key1",
        "value": "value1"
      }
    },
    {
      "property": {
        "id": "2",
        "name": "key2",
        "value": "value2"
      }
    }
  ],
  "meta": {
    "status": "SUCCESS",
    "code": "0",
    "timeStamp": "2017-05-10 22:54:02.540 +0530"
  }
}
```

### Retrieving the latest committed artifact property values for a platform instance

`GET /api/v1/platform/instances/{instance-id}/data-bag?filterBy=LAST_COMMITTED`

Response Payload
```
{
  "data": [
    {
      "property": {
        "id": "3",
        "name": "key1",
        "value": "value1"
      },
      "fromRevision": 1 // indicates that this is committed
    },
    {
      "property": {
        "id": "4",
        "name": "key2",
        "value": "value2"
      },
      "fromRevision": 1
    }
  ],
  "meta": {
    "status": "SUCCESS",
    "code": "0",
    "timeStamp": "2017-05-10 22:54:02.540 +0530"
  }
}
```

### Updating default values and descriptions for properties in the property registry

`PATCH /api/v1/metadata/property-registries/f87554da-cb8b-4b2d-b32a-0c12c84b2c81`

Request Payload

```
{
  "properties": [
    {
      "name": "first.property",
      "defaultValue": "some value"
    },
    {
      "name": "second.property",
      "description": "This is the second property",
      "defaultValue": "some other value"
    }
  ]
}
```

### Retrieving a CSV dump of Application Models

`GET /api/v1/release-management/release-pipelines/app-model-csv`

Response Payload

```
environmentName,appBlueprintName,appBlueprintVersion,propertyKey,value
CI,"First ABP",1.0,a-property,ci-1
CI,"First ABP",1.0,b-property,ci-2
CI,"First ABP",1.0,c-property,ci-3
CI,"Second ABP",1.0,a-property,ci-1
CI,"Second ABP",1.0,e-property,ci-5
CI,"Third ABP",1.0,a-property,ci-1
CI,"Third ABP",1.0,f-property,ci-6
TEST,"First ABP",1.0,c-property,test-3
TEST,"First ABP",1.0,d-property,test-4
TEST,"First ABP",1.0,a-property,test-1
TEST,"First ABP",1.0,b-property,test-2
TEST,"Second ABP",1.0,a-property,test-1
TEST,"Second ABP",1.0,e-property,test-5
TEST,"Third ABP",1.0,a-property,test-1
TEST,"Third ABP",1.0,f-property,test-6
PROD,"First ABP",1.0,a-property,prod-1
PROD,"First ABP",1.0,b-property,prod-2
PROD,"First ABP",1.0,c-property,prod-3
PROD,"First ABP",1.0,d-property,prod-4
PROD,"Second ABP",1.0,a-property,prod-1
PROD,"Second ABP",1.0,e-property,prod-5
PROD,"Third ABP",1.0,a-property,prod-1
PROD,"Third ABP",1.0,f-property,prod-6
```
 
