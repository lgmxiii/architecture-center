---
title: "How to: configure Azure governance controls for the foundational adoption stage"
description: Guidance for configuring Azure governance controls to enable a user to deploy a simple workload
author: petertay
---

# How to: design Azure governance architecture for the foundational adoption stage

The audience for this design guide is the *central IT* persona in your organization. *Central IT* is responsible for designing and implementing your organization's cloud governance architecture. As you learned in the [what is cloud resource governance?](governance-explainer.md) explainer, governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.

The goal of this guidance is to help you learn the process of designing your organization's governance architecture. To facilitate this, we'll look at a set of hypothetical goverance goals and requirements and discuss how to configure Azure's governance tools to meet them. 

In the foundational adoption stage, our goal is to deploy a simple workload to Azure. This results in the following requirements:
* Identity management for a single user or small group of users who will be responsible for deploying and maintaining the simple workload.
* Manage user identity for the simple workload as single unit.
* Manage all resources for the simple workload as a single management unit.
* A permissions model that allows for least privilege access to resources.
* Cost tracking for all resources associated with the simple workload.

## Identity management

Azure only trusts [Azure Active Directory (AD)](/azure/active-directory) to authenticate users and manage user access to resources. Azure AD users are segmented into **tenants**. A tenant is a logical construct that represents a secure, dedicated instance of Azure AD. 

Our requirement is for a single user or small group of users who will be responsible for deploying the workload, and we want to manage them as a single unit.

A single Azure AD tenant satifies these requirements. Within a single Azure AD tenant we can [create a single user or multiple users](/azure/active-directory/add-users-azure-active-directory) and [manage them as a group](/azure/active-directory/add-users-azure-active-directory).

## Resource management scope

As the number of resources deployed by your organization grows, the complexity of governing those resources grows as well. Azure implements a logical container hierarchy to enable your organization to manage your resources in groups at various levels of granularity, also known as **scope**. 

The top level of resource management scope is the **subscription** level. The next level of management scope is the **resource group** level. A resource group is a logical container for resources that enables a user or group of users to perform operations on the resources as a group. It's also important to note that every resource in Azure is deployed to a resource group. The lowest level of management scope is at the **resource** level. 

Our requirement is to manage all of the resources in the simple workload as a single unit. The first task is to select the highest scope, which in this case is a single subscription. The next task is to design the next level of scope, which is the resource group level. The considerations for resource group design are that we could create a single resource group for each resource, however, that is does not meet the requirement of managing all resources as a unit because any action on a resource would have to be repeated in each resource group. Therefore, the best design based on our requirements is to create a resource group for each workload.

## Permissions model of least privilege access 

The requirement for a permissions model that allows for least privilege access to resources means that we want users to have permissions to perfrom the task they are approved for and nothing more. As you learned in the [what is cloud governance?](governance-explainer.md) explainer, resource access permissions are assigned using role-based access control (RBAC).

RBAC roles define which capabilities the **role** has for a particular resource. For example, the built-in **contributor** role allows a user who has the role assigned to create, read, update, and delete a resource. The built-in **owner** role is similar, except it also allows the user to assign roles to other users.

A major consideration in satisfying this requirement is assigning the right role to a user at the correct resource management scope. For example, we want a *workload owner* to be able to manage Azure resources for their project but no other projects. The *workload owner* may also want to allow other users on the project team to view resources but not create, update, or delete them.

Therefore, our permissions model should include a single *central IT* user that has permission to add users to Azure AD, and permission to assign those users to a subscription. This user will have the most permissions in your organization and all permissions for other users in your organization will be granted by this *central IT* user.

When this *central IT* user creates user accounts in Azure AD, they must assign a role that to each new user that applies to all subscriptions associated with the tenant. Therefore, to meet the least privilege access requirement, your organization must make some decisions about the structure of your *workload owners* team. 

As discussed earlier, any user with the **owner** or **contributor** role at the subscription level can create, read, update, and delete any type of resource within the subscription regardless of the resource group that contains those resources. If you have a *workload owner* with the **owner** or **contributor** role at the subscription level, that *workload owner* will be able to perform all actions on a resource in any resource group within the subscription, even those for which this user might not be an owner.     

However, in order to be able to do anything at all, at least one user in the *workload owner* persona must be able to request that a resource group be created, and request that resources be deployed into that resource group. At this point, your organization must decide whether the *central IT* persona is responsible for creating resource groups, or whether the individual *workload owner* personas are trusted to not only access their own resources but the resources of each other. 

If your organization's decision is to allow only the *central IT* persona to create resource groups, the requirement of least privilege access can be satisfied by applying the **reader** role to the *workload owner* persona at the subscription level, and overriding the **reader** role with the **owner** or **contributor** role at the resource group level. 

If your organization's decision is to trust the *workload owner* persona to create resource groups at the subscription scope, the requirement of least privilege access can be satisifed by applying either the **owner** or **contributor** role.

All other users that work with the *workload owner* persona should have the least privilege access **reader** role applied at the subscription level. If the *workload owner* has the **owner** role applied at the resource group level they will have permission to change the role of the other users. Otherwise, the *workload owner* will have to request that *central IT* make any role changes.

## Cost tracking

As discussed earlier, a subscription is the top level of management scope in Azure. However, a subscription is also used to manage **cost** and apply an upper limit to the number of specific resources that can be deployed.

## Next steps

* Return to the [foundational adoption stage overview]() and learn about the different types of compute options in Azure. Then, select a type of workload and learn how to deploy it.