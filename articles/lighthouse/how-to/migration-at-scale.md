---
title: Manage Azure Migrate projects at scale
description: Azure Lighthouse helps you effectively use Azure Migrate across delegated customer resources.
ms.date: 07/16/2024
ms.topic: how-to
# Customer intent: As a service provider, I want to manage multiple Azure Migrate projects across different customer tenants using Azure Lighthouse, so that I can efficiently assess and migrate their workloads without accessing each subscription individually.
---

# Manage Azure Migrate projects at scale with Azure Lighthouse

This topic provides an overview of how [Azure Lighthouse](../overview.md) can help you use [Azure Migrate](/azure/migrate/migrate-services-overview) in a scalable way across multiple Microsoft Entra tenants.

Azure Lighthouse allows service providers to perform operations at scale across several tenants at once, making management tasks more efficient.

Azure Migrate provides a centralized hub to assess and migrate to Azure on-premises servers, infrastructure, applications, and data.

Azure Lighthouse integration with Azure Migrate lets service providers discover, assess, and migrate workloads for different customers at scale, rather than accessing each customer subscription individually. Service providers can have a single view of all of the Azure Migrate projects they manage across multiple customer tenants. Their customers have visibility into service provider actions, and they maintain control of their own environments.

> [!TIP]
> Though we refer to service providers and customers in this topic, this guidance also applies to [enterprises using Azure Lighthouse to manage multiple tenants](../concepts/enterprise.md).

Depending on your scenario, you can create the Azure Migrate project in the customer tenant or in your managing tenant. This article describes each model so you can determine which one best fits your customers' migration needs.

> [!NOTE]
> With Azure Lighthouse, partners can perform discovery, assessment and migration for on-premises VMware VMs, Hyper-V VMs, physical servers and AWS/GCP instances. For [VMware VM migration](/azure/migrate/server-migrate-overview), only the [agent-based migration method](/azure/migrate/tutorial-migrate-vmware-agent) can be used for a migration project in a delegated customer subscription. Migration using agentless replication is not currently supported through delegated access to the customer's scope.

## Create an Azure Migrate project in the customer tenant

One option when using Azure Lighthouse is to create the Azure Migrate project in the customer tenant. Users in the managing tenant can then select the customer subscription when creating a migration project. From the managing tenant, the service provider can perform the necessary migration operations. Examples of these operations are deploying the Azure Migrate appliance to discover the workloads, assessing workloads by grouping VMs and calculating cloud-related costs, reviewing VM readiness, and performing the actual migration.

In this scenario, no resources are created or stored in the managing tenant, even though the discovery and assessment steps are initiated and executed from that tenant. All of the resources, such as migration projects, assessment reports for on-premises workloads, and migrated resources at the target destination, are deployed in the delegated customer subscription. The service provider can access all customer projects from their own tenant and portal experience.

This approach minimizes context switching for service providers working across multiple customers, and lets customers keep all of their resources in their own tenants.

A high-level workflow for this model is:

1. The customer is [onboarded to Azure Lighthouse](onboard-customer.md). The Contributor built-in role is required for the identity that will be used with Azure Migrate. See the [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate) sample template for an example using this role. Before deploying the template, be sure to modify the parameter file to reflect your environment.
1. The designated user signs into the managing tenant in the Azure portal, then goes to Azure Migrate. This user [creates an Azure Migrate project](/azure/migrate/create-manage-projects), selecting the appropriate delegated customer subscription.
1. The user then [performs steps for discovery and assessment](/azure/migrate/tutorial-discover-vmware).

   For VMware VMs, before you configure the appliance, you can limit discovery to vCenter Server datacenters, clusters, a folder of clusters, hosts, a folder of hosts, or individual VMs. To set the scope, assign permissions on the account that the appliance uses to access the vCenter Server. This is useful if multiple customers' VMs are hosted on the hypervisor. You can't limit the discovery scope of Hyper-V.

    > [!NOTE]
    > For migration of VMware virtual machines, only the agent-based method is currently supported when working in a delegated customer subscription.

1. When the target customer subscription is ready, proceed with the migration through the access granted by Azure Lighthouse. The migration project containing assessment results and migrated resources are created in the customer tenant under the target subscription.

> [!TIP]
> Prior to migration, a landing zone must be deployed to provision the foundation infrastructure resources and to prepare the subscription to which virtual machines will be migrated. The Owner built-in role may be required to access or create some resources in this landing zone. Because this role is not currently supported in Azure Lighthouse, the customer may need to provide [guest access](/entra/external-id/what-is-b2b) to the service provider, or delegate admin access via the [Cloud Solution Provider (CSP) subscription model](/partner-center/customers-revoke-admin-privileges).
>
> For more information about multi-tenant landing zones, see [Considerations and recommendations for multi-tenant Azure landing zone scenarios](/azure/cloud-adoption-framework/ready/landing-zone/design-area/multi-tenant/considerations-recommendations) and the [Multi-tenant Landing-Zones demo solution](https://github.com/Azure/Multi-tenant-Landing-Zones) on GitHub.

## Create an Azure Migrate project in the managing tenant

In this scenario, the migration project and all of the relevant resources reside in the managing tenant. Customers don't have direct access to the migration project, although assessments can be shared with customers if desired. As with the previous scenario, migration-related operations such as discovery and assessment are performed by users in the managing tenant, and the migration destination for each customer is the target subscription in their tenant.

This approach enables service providers to begin migration discovery and assessment projects quickly, abstracting those initial steps from customer subscriptions and tenants.

A high-level workflow for this model is:

1. The customer is [onboarded to Azure Lighthouse](onboard-customer.md). The Contributor built-in role is required for the identity that will be used with Azure Migrate. See the [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate) sample template for an example using this role. Before deploying the template, be sure to modify the parameter file to reflect your environment.
1. The designated user signs into the managing tenant in the Azure portal, then goes to Azure Migrate. This user [creates an Azure Migrate project](/azure/migrate/create-manage-projects) in a subscription belonging to the managing tenant.
1. The user then [performs steps for discovery and assessment](/azure/migrate/tutorial-discover-vmware). The on-premises VMs are discovered and assessed within the migration project created in the managing tenant, then migrated from there.

   If you manage multiple customers in the same Hyper-V host, you can discover all workloads at once. You can select customer-specific VMs in the same group, and then create an assessment. Migration is performed by selecting the appropriate customer's subscription as the target destination. There's no need to limit the discovery scope, and you can maintain a full overview of all customer workloads in one migration project.

1. When ready, proceed with the migration by selecting the delegated customer subscription as the target destination for replicating and migrating the workloads. The new resources are created in the customer subscription, while assessment data and resources pertaining to the migration project remain in the managing tenant.

## Partner recognition for customer migrations

As a member of the [Microsoft Cloud Partner Program](https://partner.microsoft.com), you can link your partner ID with the credentials used to manage delegated customer resources. This link allows Microsoft to attribute influence and Azure consumed revenue to your organization based on the tasks you perform for customers, including migration projects.

For more information, see [Link a partner ID](/azure/cost-management-billing/manage/link-partner-id).

## Next steps

- Learn more about [Azure Migrate](/azure/migrate/migrate-services-overview).
- Learn about other [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md) supported by Azure Lighthouse.
