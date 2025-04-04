---
title: Troubleshoot Application Gateway for Containers (preview)
description: Learn how to troubleshoot common issues with Application Gateway for Containers
services: application-gateway
author: greglin
ms.service: application-gateway
ms.subservice: appgw-for-containers
ms.topic: troubleshooting
ms.date: 11/07/2023
ms.author: greglin
---

# Troubleshooting in Application Gateway for Containers (preview)

This article provides some guidance to help you troubleshoot common problems in Application Gateway for Containers.

## Find the version of ALB Controller

Before you start troubleshooting, determine the version of ALB Controller that is deployed. You can determine which version of ALB Controller is running by using the following _kubectl_ command (ensure you substitute your namespace if not using the default namespace of `azure-alb-system`):

```bash
kubectl get deployment -n azure-alb-system -o wide
```
Example output:

| NAME                     | READY | UP-TO-DATE | AVAILABLE | AGE  | CONTAINERS              | IMAGES                                                                          | SELECTOR |
| ------------------------ | ----- | ---------- | --------- | ---- | ----------------------- | ------------------------------------------------------------------------------- | -------- |
| alb-controller           | 2/2   | 2          | 2         | 18d | alb-controller           | mcr.microsoft.com/application-lb/images/alb-controller:**0.6.1**           | app=alb-controller |
| alb-controller-bootstrap | 1/1   | 1          | 1         | 18d | alb-controller-bootstrap | mcr.microsoft.com/application-lb/images/alb-controller-bootstrap:**0.6.1** | app=alb-controller-bootstrap |

In this example, the ALB controller version is **0.6.1**. 

The ALB Controller version can be upgraded by running the `helm upgrade alb-controller` command. For more information, see [Install the ALB Controller](quickstart-deploy-application-gateway-for-containers-alb-controller.md#install-the-alb-controller).

> [!Tip]
> The latest ALB Controller version can be found in the [ALB Controller release notes](alb-controller-release-notes.md#latest-release-recommended).

## Collect ALB Controller logs
Logs can be collected from the ALB Controller by using the _kubectl logs_ command referencing the ALB Controller pod.

1. Get the running ALB Controller pod name

    Run the following kubectl command. Ensure you substitute your namespace if not using the default namespace of `azure-alb-system`:
    ```bash
    kubectl get pods -n azure-alb-system
    ```
    
    You should see output similar to the following example. Pod names might differ slightly.
   
    | NAME                                     | READY | STATUS  | RESTARTS | AGE  |
    | ---------------------------------------- | ----- | ------- | -------- | ---- |
    | alb-controller-6648c5d5c-sdd9t           | 1/1   | Running | 0        | 4d6h |
    | alb-controller-6648c5d5c-au234           | 1/1   | Running | 0        | 4d6h |
    | alb-controller-bootstrap-6648c5d5c-hrmpc | 1/1   | Running | 0        | 4d6h |

    ALB controller uses an election provided by controller-runtime manager to determine an active and standby pod for high availability.
    
    Copy the name of each alb-controller pod (not the bootstrap pod, in this case, `alb-controller-6648c5d5c-sdd9t` and `alb-controller-6648c5d5c-au234`) and run the following command to determine the active pod.

    # [Linux](#tab/active-pod-linux)
    ```bash
    kubectl logs alb-controller-6648c5d5c-sdd9t -n azure-alb-system -c alb-controller | grep "successfully acquired lease"
    ```

    # [Windows](#tab/active-pod-windows)
    ```cli
    kubectl logs alb-controller-6648c5d5c-sdd9t -n azure-alb-system -c alb-controller| findstr "successfully acquired lease"
    ```
    ---

    You should see the following if the pod is primary: `successfully acquired lease azure-alb-system/alb-controller-leader-election`

2. Collect the logs

   Logs from ALB Controller will be returned in JSON format.
    
    Execute the following kubectl command, replacing the name with the pod name returned in step 1:
    ```bash
    kubectl logs -n azure-alb-system alb-controller-6648c5d5c-sdd9t
    ```
    
    Similarly, you can redirect the output of the existing command to a file by specifying the greater than (>) sign and the filename to write the logs to:
    ```bash
    kubectl logs -n azure-alb-system alb-controller-6648c5d5c-sdd9t > alb-controller-logs.json
    ```

## Configuration errors

### Application Gateway for Containers returns 500 status code

Scenarios in which you would notice a 500-error code on Application Gateway for Containers are as follows:
1. __Invalid backend Entries__ : A backend is defined as invalid in the following scenarios:
    - It refers to an unknown or unsupported kind of resource. In this case, the HTTPRoute's status has a condition with reason set to `InvalidKind` and the message explains which kind of resource is unknown or unsupported.
    - It refers to a resource that doesn't exist. In this case, the HTTPRoute's status has a condition with reason set to `BackendNotFound` and the message explains that the resource doesn't exist.
    - It refers to a resource in another namespace when the reference isn't explicitly allowed by a ReferenceGrant (or equivalent concept). In this case, the HTTPRoute's status has a condition with reason set to `RefNotPermitted` and the message explains which cross-namespace reference isn't allowed.

    For instance, if an HTTPRoute has two backends specified with equal weights, and one is invalid 50 percent of the traffic must receive a 500.

2. No endpoints found for all backends: when there are no endpoints found for all the backends referenced in an HTTPRoute, a 500 error code is obtained.

### Kubernetes Gateway resource fails to get token from credential chain

#### Symptoms

No changes to HttpRoutes are being applied to Application Gateway for Containers.

The following error message is returned on the Kubernetes Gateway resource and no changes to HttpRoutes

```YAML
status:
  conditions:
  - lastTransitionTime: "2023-04-28T22:08:34Z"
    message: The Gateway is not scheduled
    observedGeneration: 2
    reason: Scheduled
    status: "False"
    type: Scheduled
  - lastTransitionTime: "2023-04-28T22:08:34Z"
    message: "No addresses have been assigned to the Gateway : failed to get token
      from credential chain: [FromAssertion(): http call(https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/oauth2/v2.0/token)(POST)
      error: reply status code was 401:\n{\"error\":\"unauthorized_client\",\"error_description\":\"AADSTS70021:
      No matching federated identity record found for presented assertion. Assertion
      Issuer: 'https://azureregion.oic.prod-aks.azure.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/'.
      Assertion Subject: 'system:serviceaccount:azure-application-lb-system:gateway-controller-sa'.
      Assertion Audience: 'api://AzureADTokenExchange'. https://docs.microsoft.com/en-us/azure/active-directory/develop/workload-identity-federation\\r\\nTrace
      ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\\r\\nCorrelation ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\\r\\nTimestamp:
      2023-04-28 22:08:46Z\",\"error_codes\":[70021],\"timestamp\":\"2023-04-28 22:08:46Z\",\"trace_id\":\"08079978-7238-4ae3-9406-ba3b479db000\",\"correlation_id\":\"b2f10283-8dc6-4493-bb0e-b0cd009b17fb\",\"error_uri\":\"https://login.microsoftonline.com/error?code=70021\"}
      DefaultAzureCredential: failed to acquire a token.\nAttempted credentials:\n\tEnvironmentCredential:
      incomplete environment variable configuration. Only AZURE_TENANT_ID and AZURE_CLIENT_ID
      are set\n\tManagedIdentityCredential: IMDS token request timed out\n\tAzureCLICredential:
      fork/exec /bin/sh: no such file or directory]"
    observedGeneration: 2
    reason: AddressNotAssigned
    status: "False"
    type: Ready
```

#### Solution

Ensure the federated credentials of the managed identity for the ALB Controller pod to make changes to Application Gateway for Containers are configured in Azure. Instructions on how to configure federated credentials can be found in the quickstart guides:
- [Quickstart: Deploy ALB Controller](quickstart-deploy-application-gateway-for-containers-alb-controller.md#install-the-alb-controller)
