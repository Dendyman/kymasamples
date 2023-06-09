# Overview

This sample provides details on how  SAP BTP Kyma runtime can be used to extend SAP Sales Cloud (SAP Cloud for Customer).
It verifies and corrects the address data entered for the Account by using an extension built on SAP BTP Kyma runtime.

![flow](assets/flow-diagram.png)

## Prerequisites

* [SAP BTP, Kyma runtime instance](../../prerequisites/#kyma)
* [Kubernetes tooling](../../prerequisites/#kubernetes)
* [SAP Sales Cloud Tenant](https://help.sap.com/learning-journeys/04f41f184f684b84b3d8ab0b4d7c4b18)
* [SAP Sales Cloud paired with Kyma runtime](https://help.sap.com/viewer/d5fec61c279741048109d851d4d3d1ad/1908/en-US/a84a5e9266264af8ac32fe627de10bd7.html)
* `Account.Root.Created` and `Account.Root.Updated` events are registered
* [A Here project and API key created](https://developer.here.com/). Here Geocode API will be used for address correction

## Deploy

* Set up environment variables

  * OSX

    ```shell script
    export NS={your-namespace}
    ```

  * Windows PowerShell

    ```powershell
    $NS={your-namespace}
    ```

* Enable istio sidecar injection if not already done

   ```shell script
   kubectl label namespace $NS istio-injection=enabled
   ```

* Create a secret containing [here geocode api](https://developer.here.com/documentation/geocoding-search-api/dev_guide/topics/endpoint-geocode-brief.html) , credentials and central gateway url

   ![central-gw-url](assets/central-gw-url.png)

   ```shell script
   kubectl -n $NS create secret generic address-completion-here --from-literal=apiKey={your-here-api-key} --from-literal=apiUrl={here-geocode-api-url} --from-literal=centralGwUrl={your-connected-c4c-application-central-gateway-url}
   ```

* Deploy the function

   ```shell script
   kubectl -n $NS apply -f k8s/function.yaml
   ```

* Create event subscription to `Account.Root.Created` and `Account.Root.Updated` events from SAP Sales Cloud

   Update `{provide-your-application-name-as-shown-in-kyma}` and `{your-namespace}` in [k8s/subscription.yaml](k8s/subscription.yaml)

   ```shell script
   kubectl -n $NS apply -f k8s/subscription.yaml
   ```

## Verify

* Login to your SAP Sales Cloud Tenant
* Create / update a new Account in SAP Sales Cloud.
* Provide wrong address details such as a wrong pin code.
* Observe the address getting automatically corrected after few seconds.
