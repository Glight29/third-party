---


copyright:
  years: 2018
lastupdated: "2018-06-26"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Step 3: Developing and hosting your service brokers

Using the metadata you exported from the resource management console, you’ll build one or more new service brokers in the programming language of your choice.

Service brokers manage the lifecycle of services. The {{site.data.keyword.Bluemix_notm}} platform interacts with service brokers to provision and manage service instances (an instantiation of a service offering) and service bindings (The representation of an association between an application and a service instance, which often contain the credentials that the application will use to communicate with the service instance). Providing valid metadata values will create a successful RESTful API Response when a Request is performed.

You can get started building your broker by using a combination of the metadata you exported from the resource management console, our public {{site.data.keyword.Bluemix_notm}} service broker samples, and the Resource Broker API documentation. To develop your broker, you’ll:

1. View our platform provisioning scenario
2. Read through the OSB specification
2. Look at the {{site.data.keyword.Bluemix_notm}} broker samples
3. Use the Resource Broker API documentation to understand REST API endpoint logic
4. Use the metadata you exported from resource management console to inform your development
5. View the Broker information provided by the {{site.data.keyword.Bluemix_notm}} platform
6. Read through the additional recommendations to optimize your development
7. Host your broker
8. Test out your broker

## Before you begin

This step assumes that you have been approved to deliver an integrated billing service. If you haven't yet completed the initial registration and approval in Provider Workbench, see the [Getting started tutorial](/docs/third-party/index.md).
{: tip}

Ensure you have started step 1 and completed step 2
1. [Author service docs and marketing announcement](/docs/third-party/cis1-docs-marketing.html).
2. [Define your offering in the resource management console](/docs/third-party/cis2-rmc-define.html).


## View our {{site.data.keyword.Bluemix_notm}} platform provisioning scenario

You’ll be developing an Open Service Broker that works with the {{site.data.keyword.Bluemix_notm}} platform. See our [Provisioning scenario](/docs/third-party/platform.html#provisioning-scenario-pulling-it-all-together) to gain an understanding of how resource creation works.

## Become familiar with the OSB specification

{{site.data.keyword.Bluemix_notm}} uses the Open Service Broker API (OSB) `version 2.12` specification. Read through and familiarize yourself with the [Open Broker API spec](https://github.com/openservicebrokerapi/servicebroker/blob/v2.12/spec.md){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), and use the readme file as a guide to learn more.

## View our {{site.data.keyword.Bluemix_notm}} broker samples

[https://github.com/IBM/sample-resource-service-brokers](https://github.com/IBM/sample-resource-service-brokers){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

**Note:** Not all languages are represented by a sample. If you need a sample Python broker for example, you should be able to find a Cloud Foundry sample by searching Google. You may need to adjust this sample to meet the OSB requirements.


## View our {{site.data.keyword.Bluemix_notm}} Open Service Broker API Documentation

Service brokers should be developed with an understanding of the [{{site.data.keyword.Bluemix_notm}} Open Service Broker API](https://console.bluemix.net/apidocs/821-ibm-cloud-open-service-broker-api?&language=node#introduction){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"). Become familiar with the Broker API, and how it will interact with your broker or brokers.

The {{site.data.keyword.Bluemix_notm}} Open Service Broker extends the Open Service Broker 2.12 specification.
{: tip}

### Required endpoint logic for all service brokers

Service brokers must provide a standard set of metadata values that are consumed by REST APIs, and {{site.data.keyword.Bluemix_notm}} brokers must contain logic for the following REST API endpoints/paths:

<dl>
  <dt>catalog (GET)</dt>
  <dd>Returns your catalog metadata included in your broker. There are many additional catalog metadata values that are not returned - these values are added exclusively within resource management console and stored within the {{site.data.keyword.Bluemix_notm}} Catalog.</dd>
  <dt>resource instances (PUT)</dt>
  <dd>Provisions your service instance</dd>
  <dt>resource instances (DELETE)</dt>
  <dd>Deprovisions your service instance.</dd>
  <dt>resource instances (PATCH)</dt>
  <dd>Updates your service instance.</dd>
</dl>

**Note on catalog (GET)**: This endpoint defines the contract between the broker and the {{site.data.keyword.Bluemix_notm}} platform for the services and plans that the broker supports. This endpoint returns the catalog metadata stored within your broker. These values define the minimal provisioning contract between your service and the {{site.data.keyword.Bluemix_notm}} platform. All additional catalog metadata that is not required for provisioning is stored within the {{site.data.keyword.Bluemix_notm}} catalog, and any updates to catalog display values that are used to render your dashboard like links, icons, and i18n translated metadata should be updated in the resource management console, and not housed in your broker. None of metadata stored in your broker is displayed in the {{site.data.keyword.Bluemix_notm}} console or the {{site.data.keyword.Bluemix_notm}} CLI; the console and CLI will return what was set withn resource management console and stored in the {{site.data.keyword.Bluemix_notm}} catalog. These are the minimal required values that catalog (GET) should return:

```
{
       "services": [{
           "id": "ibmcloud-link",
           "name": "ibmcloud-link",
           "description": "An IBM provided service that enables aliasing to service instances in the {{site.data.keyword.Bluemix_notm}}.",
           "bindable": true,
           "plan_updateable": false,
           "plans": [
               {
                   "id": "ibmcloud-link-alias",
                   "name": "ibmcloud-alias",
                   "free": true,
                   "description": "The {{site.data.keyword.Bluemix_notm}} alias plan used for linking."
               }
               ]
       }]
}
```

### Required endpoints logic for bindable services

If your service can be bound to applications in {{site.data.keyword.Bluemix_notm}}, it must be able to return API endpoints and credentials to your service consumers. A bindable service must use the bindable operations in the Open Service Broker specification, and implement the following endpoints/paths:

<dl>
  <dt>bindings and credentials (PUT)</dt>
  <dd>Binds your service instance to an application.</dd>
  <dt>bindings and credentials (DEL)</dt>
  <dd>Unbinds your service instance from an application.</dd>
</dl>

### Required {{site.data.keyword.Bluemix_notm}} extension endpoints

The OSB specification does *not* support a disabled instance state, but not yet deleted instance state. In order for {{site.data.keyword.Bluemix_notm}} to support customers that may experience a billing lapse or other situations that result in an account suspension (but not yet cancellation), {{site.data.keyword.Bluemix_notm}} has defined extended API endpoints that allow service instances to be disabled and re-enabled. The following endpoint extensions are **required**:

<dl>
  <dt>enable and disable instances (GET)</dt>
  <dd>Status - returns the state of your service instance.</dd>
  <dt>enable and disable instances (PUT)</dt>
  <dd>Allows you to enable or disable a service instance.</dd>
</dl>

**Note**: It is the service provider's responsibility to disable access to the service instance when the disable endpoint is invoked and to re-enable that access when the enable endpoint is invoked.

## Learn how to use the exported metadata to guide your broker development

The metadata you exported from the resource management console can be used as a guide for developing your own broker. Not all of the values you entered into the resource management console are required to provision a service. The metadata that you exported from the resource management console defines the minimal provisioning contract between your service and the {{site.data.keyword.Bluemix_notm}} platform. Your exported json should provide the following values:

```
{
services :
            [
                {
                    bindable         : true,
                    description      : "Test Node Resource Service Broker Description",
                    // TODO - GUID generated by http://www.guidgenerator.com
                    // TODO - This service id must be unique within an IBM Cloud environment's set of service offerings
                    id               : "df35cab6-347b-4ba5-8f39-e9c23a237f5b",
                    metadata         :
                    {
                        displayName         : "Test Node Resource Service Broker Display Name",
                        documentationUrl    : baseMetadataUrl + "documentation.html",
                        imageUrl            : baseMetadataUrl + "services.svg", // Copied from https://github.com/carbon-design-system/carbon-icons/blob/master/src/svg/services.svg
                        instructionsUrl     : baseMetadataUrl + "instructions.html",
                        longDescription     : "Test Node Resource Service Broker Long Description",
                        providerDisplayName : "Company Name",
                        supportUrl          : baseMetadataUrl + "support.html",
                        termsUrl            : baseMetadataUrl + "terms.html"
                    },
                    name             : SERVICE_NAME,
                    // TODO - Ensure this value is accurate for your service. Requires PATCH of /v2/service_instances/:instance_id below if true
                    plan_updateable  : true,
                    tags             : ["lite", "tag1a", "tag1b"],
                    plans            :
                    [
                        {
                            bindable    : true,
                            description : "Test Node Resource Service Broker Plan Description",
                            free        : true,
                            // TODO - GUID generated by http://www.guidgenerator.com
                            // TODO - This service plan id must be unique within an IBM Cloud environment's set of service plans
                            id          : "2a1d139b-1b05-4e33-b72e-a1f8c14be559",
                            metadata    :
                            {
                                bullets     : ["Test bullet 1", "Test bullet 2"],
                                displayName : "Lite"
                            },
                            // TODO - This service plan name must be unique within the containing service definition
                            name        : "lite"
                        }
                    ]
                }
            ]
}
```


Your OSB services array must be exactly the same as the offering metadata you added to the resource management console. To ensure one to one parity between OSB and the resource management console, it is critical that you compare the services array in `catalog.json` you dowlonaded from the resource management console to the actual services array in your broker. All service and plan ids and names must match.
{: tip}

## Broker information provided by the {{site.data.keyword.Bluemix_notm}} platform

Your service broker or brokers will receive the following information from the {{site.data.keyword.Bluemix_notm}} platform:

### X-Broker-API-Originating-Identity

The **user identity header** will be provided via an API originating identity header. This request header will include the user's {{site.data.keyword.Bluemix_notm}} IAM Identity. The IAM Identity will be base64 encoded. {{site.data.keyword.Bluemix_notm}} supports a single authentication realm: `IBMid`. The `IBMid` realm uses an IBMid Unique ID (IUI) to identify the user's identity in {{site.data.keyword.Bluemix_notm}}. This IUI is an opaque string to the service provider.

Example:

```
X-Broker-API-Originating-Identity: ibmcloud eyJpYW1faWQiOiJJQk1pZC01MEdOUjcxN1lFIn0=
Decoded:
{"iam_id":"IBMid-50GNR717YE"}
```

### API header version

The **API version header** will be [2.12](https://github.com/openservicebrokerapi/servicebroker/blob/v2.12/spec.md){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"). For example: `X-Broker-Api-Version: 2.12`.

### resource instance (PUT) body.context and resource instance (PATCH) body.context

`PUT /v2/service_instances/:resource_instance_id` and `PATCH /v2/service_instances/:resource_instance_id` will receive the following value within **body.context**: `{ "platform": "ibmcloud", "account_id": "tracys-account-id", "crn": "resource-instance-crn" }`.

## Additional broker recommendations

### Recommendations on using asynchronous vs synchronous operations

The OSB API supports both synchronous and asynchronous modes of operation. If your operations are going to take less then 10 seconds, then synchronous responses are recommended.  Otherwise, you should use the asynchronous mode of operation.  More information on this is contained in the OSB specification.

If your async operation takes less than 10 seconds when trying to provision an instance, the platform will time-out.
{: tip}

### Recommendations for managing brokers across locations

It is important for users to understand the location of their cloud services for latency, availability, and data residency.

When provisioning service instances on {{site.data.keyword.Bluemix_notm}}, one of the required parameters your users will provide is the location where they want that service instance to be provisioned. Some services may support provisioning in multiple  locations. For example, a database service may support being provisioned in all {{site.data.keyword.Bluemix_notm}} regions or it may support a subset.

If your third party API-based service is implemented in another cloud and exposed into {{site.data.keyword.Bluemix_notm}}, the location should indicate the service's location in the other cloud.

When onboarding to {{site.data.keyword.Bluemix_notm}}, you must implement at least one OSB broker, but you have the option to have more then one broker depending on your deployment strategy and the locations you want to support for your service.  Within the resource management console tool, you established the mapping between your service/plan/location tuple and the broker that will service operations for that tuple. The typical choices would be to define a single broker to service all locations for your service or to define a broker per location; this choice is up to the service provider.

For a list of available locations, consult the [IBM Global Catalog Locations](https://resource-catalog.bluemix.net/search?q=kind:geography){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"). If your service requires additional locations to be defined in the Global Catalog, please consult the {{site.data.keyword.Bluemix_notm}} onboarding team.


## Host your brokers

Your broker must be hosted as part of an application that can respond to REST API calls. And your hosted location must meet {{site.data.keyword.Bluemix_notm}} security guidelines. You can be hosted in {{site.data.keyword.Bluemix_notm}}, or it can be hosted externally, so long as it is publicly accessible from {{site.data.keyword.Bluemix_notm}} itself.

To host your broker outside of IBM, you must ensure that it meets the following security guidelines:
- Must follow Transport Layer Security (TLS) protocol version 1.2
- Must be hosted on a valid HTTPs endpoint that is accessible on the public Internet

If you want to host in {{site.data.keyword.Bluemix_notm}}, you can find information about creating an app using Containers (Kubernetes) here: [Internal Adopters - Usage information](/docs/containers/cs_internal.html#cs_internal).

You will need the hosted location of your service broker to complete the next step. Have the URL and credentials associated with your app ready when moving to the next step.
{: tip}

## How to test your service's broker

You should be validating your broker by running curl commands against the different endpoints you are enabling. The sample readme provides excellent guidance for curling your OSB endpoints: https://github.com/IBM/sample-resource-service-brokers/blob/master/README.md

### How to curl your service's broker

Use the following example to test your brokers curl response:

```
curl -X PUT  https://<sample-service-broker>/v2/service_instances/<encoded-resource-crn> \
     -u '<your broker user>:<your broker password>' \
     -H 'content-type: application/json' \
      -d '{ "context": {"platform": "ibmcloud", \
                        "account_id": "34ff5928-c3c7-4d46-bbf6-1a5628c325d1", \
                        "resource_group_crn": "crn:v1:bluemix:public:resource-controller::a/003e9bc3993aec710d30a5a719e57a80::resource-group:b4570a825f7f4d57aa54e8e1d9507926", \
                        "crn": "<resource-crn>", \
                        "target_crn": "<target_crn>"}, \
            "service_id": "a07f025c-90db-4652-afd1-cf4adfac93c8", \
            "plan_id": "fe442cec-2eef-41fe-9f92-58d6c094584f"}'
```

## Next steps

You have some serious skills! You just built and hosted a service broker that meets the OSB specification. See [Step 4: Publishing and testing your service](/docs/third-party/cis4-rmc-publish.html).
