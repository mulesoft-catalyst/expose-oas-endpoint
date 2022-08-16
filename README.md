# Expose swagger/oas endpoint

This mule application downloads the OAS package from Exchange and exposes the content of the oas.json as a REST resource. The original purpose to create this mule application was to be imported in other mule applications and run ZAP (a penetration test tool that needs swagger.json to run) by passing a swagger file to it. Of course it can be used for any purpose where a swagger of the API is needed.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

- Mule 4.1.4 or above. This is mainly due to Dataweave readUrl() function that from 4.1.4 is able to read from the classpath
- The below in the settings.xml or pom.xml
```
<server>
    <id>anypoint-exchange</id>
     <username>anypoint_user</username>
    <password>pwd</password>
</server>
```
```
<repository>
    <id>anypoint-exchange</id>
<name>Anypoint Exchange</name>
    <url>https://maven.anypoint.mulesoft.com/api/v2/maven</url>
    <layout>default</layout>
</repository>
```

```
Give examples
```

### Installing

- Get this example and generate a jar from it (export jar in studio) and install it into a maven repo (coordinates as below).

```<groupId>com.mulesoft</groupId>
<artifactId>expose-oas-endpoint</artifactId>
<version>1.0.0</version>```
```
- Add the maven dependency in your application pom
```
mvn install:install-file -Dfile=expose-oas-endpoint.jar -DgroupId=com.mulesoft -DartifactId=expose-oas-endpoint -Dversion=1.0.0 -Dpackaging=jar
```
- Create a file named src/main/resources/configurations.yaml and in there specify the swagger file path, the base path and the other properties according to needs (see its content below)

      # This represents the swagger file present on Exchange
      swagger_file_name: api.json
      orgId: 64fe4806-309d-4611-90dd-XXXX
      #These two below are specific to the application being implemented and MUST be changed. APIName is also referred to as artifactId and can be retrieved from the Exchange page -> Download -> As OAS -> open the exchange.json
      apiName: collection-core-apis
      version: 1.0.21
      dependency_type: zip
      classifier: oas
      # This is to modify the basePath property in the returned swagger file
      basePath: /api
      # This is to specify the path where to save the swagger file
      swagger_file_modified_path: .
      #This is the file produced name, created <MULE_HOME>/bin
      swagger_file_modified_name: swagger.json
      #This allows to specify the endpoint e.g. <baseURI>/swagger.json
      swagger_http_endpoint: swagger.json
      #Make sure this property has the same value as the http listener config in your app
      http_listener_config: HTTP_Listener_config
      swagger_flow_initial_status: stopped

In the global.xml of the application create a global configuration "Configuration properties" and specify configurations.yaml as text. This will tell the application to replace the properties ${...} used in the XML with what written in the configurations.yaml file.
In the global.xml of the application create a global configuration "Import" and specify expose-oas-endpoint.xml, this will import the flow that manages and exposes the oas file, available at /swagger.json by default, but amendable in the properties.
Make sure to set the new flow (e.g. get:\swagger.json) initial state as "stopped", there is a property for this too. This will be started on-demand and never on PROD
Start the get:\swagger.json flow from Runtime Manager or see how to with ARM APIs at the end of this page


### ZAP penetration testing tool

To perform a penetration scan with ZAP one can use the below docker run script, appropriately customised for the current environment:

> docker run -v /<your_own_volume_path>:/zap/wrk/:rw -t owasp/zap2docker-weekly zap-api-scan.py -t [http://host.docker.internal:8081/swagger.json](http://host.docker.internal:8081/swagger.json) -f openapi -d -r api-scan-report.html -w api-scan.md

The scan should produce two reports files, api-scan-report.html and api-scan.md which will describe the outcome of the test.

### ARM APIs to stop/start flows call

To retrieve the list of applications and from them get to a specific flow id to be able to start and stop its execution. [ARM API Spec](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-XXXXXXX.anypoint-platform/arm-rest-services/1.0.7/console/method/#2122/)

- GET Applications: From the json select the specific application id needed

```
curl '[https://anypoint.mulesoft.com/hybrid/api/v1/applications](https://anypoint.mulesoft.com/hybrid/api/v1/applications)' \

-X GET \

-H 'x-anypnt-org-id:64fe4806-309d-4611-90dd-XXXXX' \

-H 'authorization: Bearer XXXXXX-fd6d-4a49-85f6-XXXX' \

-H 'x-anypnt-env-id:976b54e8-1a94-4678-828b-XXX'
```

- GET Flows: For a given application get a list of flows and pick the id with the specific name e.g. get:\swagger.json

```
curl '[https://anypoint.mulesoft.com/hybrid/api/v1/applications/2821490/flows](https://anypoint.mulesoft.com/hybrid/api/v1/applications/2821490/flows)' \

-X GET \

-H 'x-anypnt-org-id:64fe4806-309d-4611-90dd-XXXXX' \

-H 'authorization: Bearer 0475ceea-fd6d-4a49-85f6-XXXXXX' \

-H 'x-anypnt-env-id:976b54e8-1a94-4678-828b-XXXX'
```

- PATCH Flow: Use the flodId retrieved above to execute a start or a stop

```
curl [https://anypoint.mulesoft.com/hybrid/api/v1/applications/2821490/flows/10505332](https://anypoint.mulesoft.com/hybrid/api/v1/applications/2821490/flows/10505332) \

-X PATCH \

-H 'x-anypnt-org-id:64fe4806-309d-4611-90dd-XXXXX' \

-H 'authorization: Bearer 0475ceea-fd6d-4a49-85f6-XXXXX' \

-H 'x-anypnt-env-id:976b54e8-1a94-4678-828b-XXXXXX' \

-H 'content-type: application/json' \

-d '[{"id":10505332,"desiredStatus":"STARTED"}]'
```

## Running the tests

There is a MUnit test but in order to run it successfully one needs to amend the below maven properties and

    <api.orgid>64fe4806-309d-4611-90dd-XXXXXX</api.orgid>
    <api.name>collection-core-apis</api.name>
    <api.artifact.version>1.0.21</api.artifact.version>

and also the relevant ones in the configurations.yaml file

    orgId: 64fe4806-309d-4611-90dd-XXXXXX
    apiName: collection-core-apis
    version: 1.0.21

### And coding style tests

Explain what these tests test and why

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Maven](https://maven.apache.org/) - Dependency Management


## Authors

* **Ettore Giallaurito** - *Initial work* -

See also the list of [contributors]
* **Marcel Bakker**
