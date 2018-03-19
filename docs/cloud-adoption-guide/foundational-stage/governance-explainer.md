---
title: "Explainer: what is cloud goverance?"
description: Explains the concept of resource governance for Azure and cloud
author: petertay
---

# Explainer: what is cloud resource governance?

In the [how does Azure work](azure-explainer.md) explainer, you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users. Azure enables an organization's development and IT departments agility by making it easy to create, read, update, and delete resources as needed.

While giving unrestricted access to developers to create resources can make them very agile, it can also lead to unintended cost consequences. For example, a development team can deploy virtual networking and machines for approved testing purposes and forget to delete them after the testing phase is complete. The consequence of these forgotten resources is that costs continue to accrue after the testing phase, even though the resource usage is no longer approved. 

The solution to this problem is resource access **governance**. Governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of an organization. These goals and requirements are unique to each organization so it's not possible to have a one-size-fits-all approach to governance. Azure provides a set of governance tools, and it is up to the organization to decide the best way to configure them.

Azure implements two primary governance tools: **resource based access control (RBAC)**, and **resource policy**. 

RBAC roles are assigned to users or groups of users, and define which capabilities of the role for a particular resource. For example, the **owner** role enables all capabilites (create, read, update, and delete) for a resource, while the  **reader** roles enables only the read capability. Roles can be defined with a broad scope that applies to many resources types, or a narrow scope that applies to a few. 

Resource policies defines rules for resource creation. For example, a resource policy can limit the SKU of a VM to a particular pre-appproved size. Or, a resource policy can enforce the addition of a tag with a cost center when the request is made to create the resource. 

The combination of these define which users in your organization have permission to perform operations on a policy-restricted set of resources.  

When configuring these tools, an important consideration is organizational agility. As access to resources is more "locked down", the more manual steps are required for your developers and IT workers to gain access to them. For example, if your organization decides to allow only a single person to create resources in Azure, all requests must go through that single person. That single person has finite capabilities and may become backlogged. This will result in unproductive teams waiting for their resources to be created and unneeded resources accruing costs while they wait to be deleted. 

A better balance between agility and governance is possible by implementing self-service, managed and monitored by Azure governance tools. This enables your developers and IT to manage their resources in an agile manner while ensuring that policies are complied with and costs are controlled.

## Scope of governance

Some organizations will have thousands of resources in Azure owned by multiple teams across multiple departments. Azure provides a hierarchichal set of logical groups that organizations can use to govern resources in sets. 

Resources can be grouped to one of two scopes: either the parent **subscription** level or the child **resource group** level. User identity is stored in an Azure Active Directory (AD) **tenant** and is associated at the subscription level. Therefore, configuring your organization's governance controls is organization users into logical groups in your Azure tenant, creating appropriate roles and policy for restricing access to Azure resources, and mapping the two together.