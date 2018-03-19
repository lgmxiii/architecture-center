---
title: "Guidance: Azure Active Directory Design Guide"
description: Intermediate stage identity management guidance for enterprises adopting Azure  
author: petertay
---

# Guidance: Azure Active Directory Design Guide

In the foundational adoption stage, you learned that Azure trusts Azure Active Directory (Azure AD) to provide authentication for users that want to create, read, update, or delete Azure resources. You learned that for a single PaaS or IaaS workload, a single Azure AD instance was sufficient because it's assumed that a single team will be responsible for that workload and all users can be created in that instance. 

In reality, organizations will have multiple teams responsible for multiple [workloads](workload-explainer.md) in Azure and will need to manage the identity of the individuals on those teams in different ways. 

These teams may have different functions such as developing customer facing applications or developing internal line of business apps. And each of these teams will most likely have different requirements for managing resources in Azure. The team developing the customer facing application might have a requirement to manage customer user identity, so they must be able to deploy an instance of Azure Active Directory. The team developing the internal line of business application will have a requirement to make use of existing on-premises user identities, so they do not need to actually provision an instance of Azure AD, but they may require synchronization or federation from on-premises to Azure AD.

In addition to these functional requirements, the organization will further have operational requirements such as security, cost management, and monitoring. For example, the organization may have different trust levels per team: a team working with high business impact (HBI) data might be more trusted than a team working with low business impact (LBI) data.

In addition, an organization may have office in multiple geographical regions, have multiple subsidiaries, and may divest internal divisions to external organizations. The composition of departments and teams will change over time as people move into, around, and out of the organization.

Managing a large number of individuals and a large number of workloads over time can be a very complex and dynamic task. The place to start this task is by deciding your organization's Azure AD tenant design.

> [NOTE]
> This guidance covers Azure AD design for on-premises users only. To learn more about using Azure AD for your customers, review the [Azure AD B2C documentation](/azure/active-directory-b2c). To learn more about using Azure AD for your partners, review the [Azure AD B2B docmentation](/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b).

## Design Considerations

There are two primary considerations when deciding on an Azure AD tenant design for governance of Azure resources. First, the number of tenants, and second, selecting the type of tenant required from the multiple editions of Azure AD based on feature requirements. 

Let's take a look the considerations when deciding the number of tenants first:

* Segmenting users into a large number of tenants results in less users to manage within each tenant. This means that you will need a system to determine where a user's account is stored, and you will need additional functionality for aggregation of user activity monitoring from each tenant for security purposes. Users that require access to resources spread across multiple Azure AD tenants will have multiple identities: one in each tenant.  
* Segmenting users into less tenants results in more users per tenant. This means that you will need to utilize groups to organize users into manageable sizes. User activity monitoring can be done more easily because multiple reports do not need to be aggregated.
* RBAC and policy are scoped to the subscription or resource group level, so users that you want to share common resource policy and common roles are required to be stored in the same Azure AD tenant. It is possible to synchronize user objects between Azure AD tenants, therefore it is also not possible to synchronize the resource policy and RBAC roles assignments in each Azure AD tenant.
* Azure AD tenants are associated with a region. There are several considerations for choosing to create a separate Azure AD instance for users in multiple regions:
    * Your organization may have security and compliance issues that differ by region for your users. In this case, you may require a separate Azure AD tenant for each region. For example, if you have users in Germany, you must deploy Azure AD in Azure Germany.
    * If your users are located in a region that is a significant distance from the region where the Azure AD tenant is located, they may experience latency when authenticating. 

Deciding on the correct number and size of Azure AD tenants begins with determing the highest priority common criteria for grouping together users. There are many criteria to use to group users, for example users that require access to:
* Azure resources, 
* Office 365,
* on-premises line of business applications,
* applications from the Azure application gallery, or,
* applications offered by partners.
Based on your organization's business requirements, you may choose to group by one or many of these criteria. You can combine criteria, select the users that meet those multiple criteria, and then include those users in an Azure AD tenant. 

Once you have decided upon the right number number of Azure AD tenants, the next step is to select the Azure AD edition that provides the features you require.

* does your organization have an existing Office 365 enrollment? 
    * yes: your organization already has an existing instance of Azure AD, Office 365 edition. Depending on which identity model you selected when you set up your Office 365 account, you may or may not wish to upgrade to an 

* do you have an existing on-premises Active Directory Domain Services with user accounts that you want to use for identity in Azure?
    * yes
        * do you have an existing Office 365 account?
            * yes: depending on the identity model you selected when you set up your Office 365 account, you may already have synchronized or federated your organization's user identity to Azure AD. 
            * no:
                * do you want to continue to store identity **only** on-premises?
                    * yes - use federation
                    * no - use synchronization
    * no: any Azure AD is suitable.

* what protocols do your on-premises applications need to support?
    * SAML/WS-FED/WS-TRUST: supported by all Azure AD editions.
    * OpenID Connect/OAuth 2.0: supported by all Azure AD skus
    * any other protocols: not supported by Azure AD

* will you be using single sign-on for your all of your applications, including applications from the application gallery, and partner applications?
    * yes: will each user use SSO for less than 10 applications?
        * yes: the free, basic, or O365 edition.
        * no: P1 or P2 edition.
    * no: any edition.
