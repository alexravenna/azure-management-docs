---
title: Create a SQL Managed Instance enabled by Azure Arc
description: Deploy SQL Managed Instance enabled by Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-sql-mi
ms.custom: devx-track-azurecli
author: AbdullahMSFT
ms.author: amamun
ms.reviewer: mikeray
ms.date: 07/30/2021
ms.topic: how-to
# Customer intent: As a database administrator, I want to deploy a SQL Managed Instance using Azure Arc, so that I can manage my SQL databases in a consistent manner across hybrid cloud environments.
---

# Deploy a SQL Managed Instance enabled by Azure Arc

[!INCLUDE [azure-arc-common-prerequisites](./includes/azure-arc-common-prerequisites.md)]

To view available options for the create command for SQL Managed Instance enabled by Azure Arc, use the following command:

```azurecli
az sql mi-arc create --help
```

To create a SQL Managed Instance enabled by Azure Arc, use `az sql mi-arc create`. See the following examples for different connectivity modes:

> [!NOTE]
>  A ReadWriteMany (RWX) capable storage class needs to be specified for backups. Learn more about [access modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

If no storage class is specified for backups, the default storage class in Kubernetes is used and if this is not RWX capable, the SQL Managed Instance enabled by Azure Arc installation may not succeed.

### [Directly connected mode](#tab/directly-connected-mode)

```azurecli
az sql mi-arc create --name <name> --resource-group <group> -–subscription <subscription>  --custom-location <custom-location> --storage-class-backups <RWX capable storageclass>
```

Example:

```azurecli
az sql mi-arc create --name sqldemo --resource-group rg -–subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  --custom-location private-location --storage-class-backups mybackups
```


### [Indirectly connected mode](#tab/indirectly-connected-mode)

```azurecli
az sql mi-arc create -n <instanceName> --storage-class-backups <RWX capable storageclass>  --k8s-namespace <namespace> --use-k8s
```

Example:

```azurecli
az sql mi-arc create -n sqldemo --storage-class-backups mybackups --k8s-namespace my-namespace --use-k8s
```


---

> [!NOTE]
> Names must be less than 60 characters in length and conform to [DNS naming conventions](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#rfc-1035-label-names).
> When specifying memory allocation and vCore allocation use this formula to ensure your performance is acceptable: for each 1 vCore you should have at least 4GB of RAM of capacity available on the Kubernetes node where the SQL Managed Instance enabled by Azure Arc pod will run.
> If you want to automate the creation of SQL Managed Instance enabled by Azure Arc and avoid the interactive prompt for the admin password, you can set the `AZDATA_USERNAME` and `AZDATA_PASSWORD` environment variables to the desired username and password prior to running the `az sql mi-arc create` command.
> If you created the data controller using AZDATA_USERNAME and AZDATA_PASSWORD in the same terminal session, then the values for AZDATA_USERNAME and AZDATA_PASSWORD will be used to create the SQL Managed Instance enabled by Azure Arc too.
> [!NOTE]
> If you are using the indirect connectivity mode, creating SQL Managed Instance enabled by Azure Arc in Kubernetes will not automatically register the resources in Azure. Steps to register the resource are in the following articles: 
> - [Upload billing data to Azure and view it in the Azure portal](view-billing-data-in-azure.md) 
> 


## View instance on Azure Arc

To view the instance, use the following command:

```azurecli
az sql mi-arc list --k8s-namespace <namespace> --use-k8s
```

You can copy the external IP and port number from here and connect to SQL Managed Instance enabled by Azure Arc using your favorite tool for connecting to eg. SQL Server or Azure SQL Managed Instance such as Azure Data Studio or SQL Server Management Studio.

[!INCLUDE [use-insider-azure-data-studio](includes/use-insider-azure-data-studio.md)]

## Related content
- [Connect to SQL Managed Instance enabled by Azure Arc](connect-managed-instance.md)
- [Register your instance with Azure and upload metrics and logs about your instance](upload-metrics-and-logs-to-azure-monitor.md)
- [Create SQL Managed Instance enabled by Azure Arc using Azure Data Studio](create-sql-managed-instance-azure-data-studio.md)

