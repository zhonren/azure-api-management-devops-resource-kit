# Table of Contents

1. [ Creator ](#Creator)
    * [Create the Config File](#creator1)
    * [Running the Creator](#creator2)
2. [ Extractor ](#Extractor)
    * [Running the Extractor](#running-the-extractor)

# Creator

This utility creates [Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) for an API based on the [OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification) of the API. Optionally, you can provide policies you wish to apply to the API and its operations in seperate files.

<a name="creator1"></a>

## Create the Config File

The utility requires one argument, --configFile, which points to a yaml file that controls the ARM templates generated by the Creator tool. The file contains a Creator Configuration object whose schema and related schemas are listed below:

#### Creator Configuration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| version               | string                | Yes                   | Configuration version.                            |
| apimServiceName       | string                | Yes                   | Name of the APIM service to deploy resources into.    |
| apiVersionSets         | Array<[APIVersionSetConfiguration](#APIVersionSetConfiguration)> | No               | List of API Version Set configurations.                        |
| apis                   | Array<[APIConfiguration](#APIConfiguration)>      | Yes                   | List of API configurations.                                |
| products                   | Array<[ProductConfiguration](#ProductConfiguration)>      | No                   | List of Product configurations.                                |
| loggers                   | Array<[LoggerConfiguration](#LoggerConfiguration)>      | No                   | List of Logger configurations.                                |
| authorizationServers                   | Array<[AuthorizationServerContractProperties](#https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/authorizationservers#AuthorizationServerContractProperties)>      | No                   | List of Authorization Server configurations.                                |
| backends                   | Array<[BackendContractProperties](#https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/backends#BackendContractProperties)>      | No                   | List of Backend configurations.                                |
| outputLocation        | string                | Yes                   | Local folder the utility will write templates to. |
| linked                | boolean               | No                    | Determines whether the utility should create a master template that links to all generated templates. |
| linkedTemplatesBaseUrl| string                | No                    | Location that stores linked templates. Required if 'linked' is set to true. |

#### APIConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| name                  | string                | Yes                   | API identifier. Must be unique in the current API Management service instance.                                 |
| description           | string                | No                    | Description of the API.                          |
| serviceUrl            | string                | No                    | Absolute URL of the backend service implementing this API.                                 |
| type                  | enum                  | No                    | Type of API. - http or soap                      |
| openApiSpec           | string                | Yes                   | Location of the Open API Spec file. Can be url or local file.                          |
| policy                | string                | No                    | Location of the API policy XML file. Can be url or local file.                          |
| suffix                | string                | Yes                    | Relative URL uniquely identifying this API and all of its resource paths within the API Management service instance. It is appended to the API endpoint base URL specified during the service instance creation to form a public URL for this API.                       |
| subscriptionRequired  | boolean               | No                    | Specifies whether an API or Product subscription is required for accessing the API.                         |
| isCurrent             | boolean               | No                    | Indicates if API revision is current api revision.    |
| apiVersion            | string                | No                    | Indicates the Version identifier of the API if the API is versioned.                         |
| apiVersionDescription | string                | No                    | Description of the API Version.                   |
| apiRevision           | string                | No                    | Describes the Revision of the API. If no value is provided, default revision 1 is created.                  |
| apiRevisionDescription| string                | No                    | Description of the Api Revision.                 |
| apiVersionSetId       | string                | No                    | A resource identifier for the related ApiVersionSet. Value must match the resource id on an existing version set and is irrelevant if the apiVersionSet property is supplied.       |
| operations            | Dictionary<string, [APIOperationPolicyConfiguration](#APIOperationPolicyConfiguration)> | No    | XML policies that will be applied to operations within the API. Keys must match the operationId property of one of the API's operations.                 |
| authenticationSettings| [AuthenticationSettingsContract](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2018-06-01-preview/service/apis#AuthenticationSettingsContract)                | No                    | Collection of authentication settings included into this API.                         |
| products              | string                | No                    | Comma separated list of existing products to associate the API with.                   |
| protocols             | string                | No                    | Comma separated list of protocols used between client and APIM service.                   |
| diagnostic            | [APIDiagnosticConfiguration](#APIDiagnosticConfiguration) | No | Diagnostic configuration. |

#### APIOperationPolicyConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| policy                | string                | Yes                    | Location of the operation policy XML file. Can be url or local file.      |

#### APIDiagnosticConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| name                  | enum                | No                    | Name of API Diagnostic - azureEventHub or applicationInsights       |

_Additional properties found in [DiagnosticContractProperties](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/apis/diagnostics#DiagnosticContractProperties)_

#### APIVersionSetConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| id                    | string                | No                    | ID of the API Version Set.                        |

_Additional properties found in [ApiVersionSetContractProperties](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/apiversionsets#ApiVersionSetContractProperties)_

#### ProductConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| policy                | string                | No                    | Location of the Product policy XML file. Can be url or local file.                          |

_Additional properties found in [ProductContractProperties](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/products#ProductContractProperties)_

#### LoggerConfiguration

| Property              | Type                  | Required              | Value                                            |
|-----------------------|-----------------------|-----------------------|--------------------------------------------------|
| name                  | string                | Yes                   | Name of the Logger                         |

_Additional properties found in [LoggerContractProperties](https://docs.microsoft.com/en-us/azure/templates/microsoft.apimanagement/2019-01-01/service/loggers#LoggerContractProperties)_

### Sample Config File

The following is a full config.yml file with each property listed:

```
version: 0.0.1
apimServiceName: myAPIMService
apiVersionSets:
    - id: myAPIVersionSetID
      displayName: swaggerPetstoreVersionSetLinked
      description: a description
      versioningScheme: Query
      versionQueryName: versionQuery
      versionHeaderName: versionHeader
    - id: secondAPIVersionSetID
      displayName: secondSet
      description: another description
      versioningScheme: Header
      versionQueryName: versionQuery
      versionHeaderName: versionHeader
apis:
    - name: myAPI
      type: http
      description: myFirstAPI
      serviceUrl: http://myApiBackendUrl.com
      openApiSpec: C:\Users\myUsername\Projects\azure-api-management-devops-example\src\APIM_ARMTemplate\apimtemplate\Creator\ExampleFile\OpenApiSpecs\swaggerPetstore.json
      policy: C:\Users\myUsername\Projects\azure-api-management-devops-example\src\APIM_ARMTemplate\apimtemplate\Creator\ExampleFiles\XMLPolicies\apiPolicyHeaders.xml
      suffix: conf
      subscriptionRequired: true
      isCurrent: true
      apiVersion: v1
      apiVersionDescription: My first version
      apiVersionSetId: myAPIVersionSetID
      apiRevision: 1
      apiRevisionDescription: My first revision
      operations:
        addPet:
          policy: C:\Users\myUsername\Projects\azure-api-management-devops-example\src\APIM_ARMTemplate\apimtemplate\Creator\ExampleFile\XMLPolicies\operationRateLimit.xml
        deletePet:
          policy: C:\Users\myUsername\Projects\azure-api-management-devops-example\src\APIM_ARMTemplate\apimtemplate\Creator\ExampleFile\XMLPolicies\operationRateLimit.xml
      products: starter, platinum
      authenticationSettings:
        oAuth2:
            authorizationServerId: myAuthServer
            scope: myScope
      diagnostic:
        name: applicationinsights
        alwaysLog: allErrors
        loggerId: myAppInsights
        sampling:
          samplingType: fixed
          percentage: 50
        frontend: 
          request:
            headers:
            body: 
              bytes: 512
          response: 
            headers:
            body: 
              bytes: 512
        backend: 
          request:
            headers:
            body: 
              bytes: 512
          response: 
            headers:
            body: 
              bytes: 512
        enableHttpCorrelationHeaders: true
products:
    - displayName: platinum
      description: a test product
      terms: some terms
      subscriptionRequired: true
      approvalRequired: true
      subscriptionsLimit: 1
      state: notPublished
      policy: C:\Users\myUsername\Projects\azure-api-management-devops-example\src\APIM_ARMTemplate\apimtemplate\Creator\ExampleFile\XMLPolicies\productSetBodyBasic.xml
loggers:
    - name: myAppInsights
      loggerType: applicationInsights
      description: a test app insights
      credentials:
        instrumentationKey: 45d4v88-fdfs-4b35-9232-731d82d4d1c6
      isBuffered: true
authorizationServers:
    - displayName: myAuthServer
      description: test server
      clientRegistrationEndpoint: https://www.contoso.com/apps
      authorizationEndpoint: https://www.contoso.com/oauth2/auth
      authorizationMethods:
        - GET
      tokenEndpoint: https://www.contoso.com/oauth2/token
      supportState: true
      defaultScope: read write
      grantTypes:
        - authorizationCode
        - implicit
      bearerTokenSendingMethods:
        - authorizationHeader
      clientId: 1
      clientSecret: 2
      resourceOwnerUsername: un
      resourceOwnerPassword: pwd
backends:
    - title: myBackend
      description: description5308
      url: https://backendname2644/
      protocol: http
      credentials:
        query: 
          sv: 
            - xx
            - bb
        header: 
          x-my-1:
            - val1
            - val2
        authorization: 
          scheme: Basic
          parameter: opensesma
      proxy:
        url: http://192.168.1.1:8080
        username: Contoso\admin
        password: opensesame
      tls:
        validateCertificateChain: false
        validateCertificateName: false
outputLocation: C:\Users\myUsername\GeneratedTemplates
linked: false
linkedTemplatesBaseUrl : https://mystorageaccount.blob.core.windows.net/mycontainer
```

<a name="creator2"></a>

## Running the Creator
Below are the steps to run the Creator from the source code:

- Clone this repository and restore its packages using ```dotnet restore```
- Navigate to {path_to_folder}/src/APIM_ARMTemplate/apimtemplate directory
- Run the following command:
```dotnet run create --configFile CONFIG_YAML_FILE_LOCATION ```

You can also run it directly from the [releases](https://github.com/Azure/azure-api-management-devops-resource-kit/releases).

# Extractor

This utility generates [Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) by extracing existing configurations of one or more APIs in an API Management instance. 

<a name="prerequisite"></a>

## Prerequisite

To be able to run the Extractor, you would first need to [install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).

<a name="extractor1"></a>

## Running the Extractor
Below are the steps to run the Extractor from the source code:
- Clone this repository and navigate to {path_to_folder}/src/APIM_ARMTemplate/apimtemplate 
- Restore its packages using ```dotnet restore```
- Make sure you have signed in using Azure CLI and have switched to the subscription containing the API Management instance from which the configurations will be extracted. 
```
az login
az account set --subscription <subscription_id>
```

#### Extractor Arguments

| Property              | Required              | Value                                             |
|-----------------------|-----------------------|---------------------------------------------------|
| sourceApimName        | Yes                   | Name of the source APIM instance.                 |
| destinationApimName   | Yes                   | Name of the destination APIM instance.            |
| resourceGroup         | Yes                   | Name of the resource group.                       |
| fileFolder            | Yes                   | Path to output folder                             |
| apiName               | No                    | Name of API. If provided, Extractor executes single API extraction. Otherwise, Extractor executes full extraction.  Note:  This is the "Name" value as seen in the API settings, not "Display Name" and is case sensitive.     |
| linkedTemplatesBaseUrl| No                    | Linked templates remote location. If provided, Extractor generates master template and requires linked templates pushed to remote location.                                   |
| policyXMLBaseUrl      | No                    | Policy XML files remote location. If provided, Extractor generates policies folder with xml files, and requires they be pushed to remote location.                              |

To run the Extractor with all arguments (executing a single API extraction with linked templates and policy file generation), use the following command: 
```
dotnet run extract --sourceApimName <name_of_the_source_APIM_instance> --destinationApimName <name_of_the_destination_APIM_instance> --resourceGroup <name_of_resource_group> --fileFolder <path_to_folder> --apiName <api_name> --linkedTemplatesBaseUrl <linked_templates_remote_location> --policyXMLBaseUrl <policies_remote_location>
```

You can also run it directly from the [releases](https://github.com/Azure/azure-api-management-devops-resource-kit/releases).