---
title: 'Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service clusters and node pools (Preview) '
description: Learn how to enable Azure Linux 3.0 for AKS clusters and node pools.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: references_regions, devx-track-azurecli, linux-related-content
ms.topic: quickstart
ms.date: 10/10/2024

---
# Quickstart: Enable Azure Linux 3.0 for Azure Kubernetes Service (AKS) clusters and node pools (Preview)

This article shows you how to enable Azure Linux 3.0 as the default Azure Linux node OS for new AKS clusters and node pools running AKS version 1.31. Once enabled, any new AKS clusters or node pools created with the `--os-sku=AzureLinux` option defaults to using Azure Linux 3.0.

## Limitations

* Not supported on Kubernetes version 1.30 and below.
* Existing clusters or node pools running Azure Linux 2.0 can't be upgraded to 3.0. New node pools or clusters need to be created. 
* Azure Linux 3.0 FIPS image is in preview and is FIPS compliant but not verified as the crypto modules are [Modules in Process with NIST](https://csrc.nist.gov/Projects/Cryptographic-Module-Validation-Program/Modules-In-Process/IUT-List).
* **Supported Regions**: Azure Linux 3.0 support is in preview as part of the `v20241025` release. Visit the [AKS Release Tracker](https://releases.aks.azure.com/) for the latest on which regions are on this release. 

## Enable Azure Linux 3.0 as default for new clusters and nodepools

[!INCLUDE [preview features callout](~/reusable-content/ce-skilling/azure/includes/aks/includes/preview/preview-callout.md)]

1. Register the `AzureLinuxV3Preview` feature flag using the [`az feature register`](/cli/azure/feature#az-feature-register) command.  

    ```azurecli-interactive  
     az feature register --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"  
    ```  

    It takes a few minutes for the status to show *Registered*.  

1. Verify the registration status using the [`az feature show`](/cli/azure/feature#az-feature-show) command.  

    ```azurecli-interactive
    az feature show --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"
    ```

1. Once these steps are completed, you can deploy the cluster using the method of your choice to use Azure Linux 3.0 as the node OS:

    - [Quickstart with CLI](./quickstart-azure-cli.md)
    - [Quickstart with PowerShell](./quickstart-azure-powershell.md)
    - [Quickstart with Terraform](./quickstart-terraform.md)
    - [Quickstart with Azure Resource Manager (ARM)](./quickstart-azure-resource-manager-template.md)

## Disable Azure Linux 3.0
1. Unregister the `AzureLinuxV3Preview` feature flag using the [`az feature unregister`](/cli/azure/feature#az-feature-unregister) command. 
    ```azurecli-interactive
    az feature unregister --namespace "Microsoft.ContainerService" --name "AzureLinuxV3Preview"
    ```
1. Refresh the registration of the *Microsoft.ContainerService* resource provider using the [`az provider register`](/cli/azure/provider#az-provider-register) command.
    ```azurecli-interactive
    az provider register --namespace "Microsoft.ContainerService"
    ```
## Next steps
For more information about Azure Linux 3.0 Preview, see [What's new with Azure Linux 3.0?](./intro-azure-linux.md).