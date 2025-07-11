---
author: chasedmicrosoft
ms.service: azure-container-registry
ms.topic: include
ms.date: 10/07/2021
ms.author: doveychase
# Customer intent: "As a cloud administrator, I want to retrieve the registry connection string and related settings, so that I can configure and manage my container registries effectively."
---
Command output includes the registry connection string and related settings. The following example output shows the connection string for the connected registry named *myconnectedregistry* with parent registry *contosoregistry*:

```json
{
  "ACR_REGISTRY_CONNECTION_STRING": "ConnectedRegistryName=myconnectedregistry;SyncTokenName=myconnectedregistry-sync-token;SyncTokenPassword=xxxxxxxxxxxxxxxx;ParentGatewayEndpoint=contosoregistry.eastus.data.azurecr.io;ParentEndpointProtocol=https"
}
```
