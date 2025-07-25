---
title: include file
description: include file
services: lighthouse
author: JnHs
ms.service: azure-lighthouse
ms.topic: include
ms.date: 07/10/2024
ms.author: jenhayes
# Customer intent: As a service provider managing resources for multiple customers, I want to implement cross-tenant management solutions using deployment templates, so that I can efficiently create and configure resources across different Azure tenants.
---

These samples illustrate various tasks that can be performed in cross-tenant management scenarios.

| **Template** | **Description** |
|---------|---------|
| [`create-keyvault-secret`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/create-keyvault-secret) | Creates a Key Vault in the customer's tenant and creates access policies.
| [`cross-rg-deployment`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/cross-rg-deployment) | Deploys storage accounts into two different resource groups.|
| [`deploy-azure-mgmt-services`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-mgmt-services) | Creates Azure management services, links them together, and deploys solutions. For an end-to-end deployment, use the [rgWithAzureMgmt.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/deploy-azure-mgmt-services/rgWithAzureMgmt.json) template. |
| [`deploy-azure-security-center`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-security-center) | Enables and configures Microsoft Defender for Cloud within the targeted Azure subscription. |
| [`deploy-azure-sentinel`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-azure-sentinel) | Deploys and enables Microsoft Sentinel on an existing Log Analytics workspace in a delegated subscription. |
| [`deploy-log-analytics-vm-extensions`](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/deploy-log-analytics-vm-extensions) | Allows you to deploy Log Analytics VM extensions to your Windows and Linux VMs, connecting them to the designated Log Analytics workspace. |
