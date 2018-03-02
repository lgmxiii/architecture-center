---
title: "Explainer: ?"
description: Explains how 
author: petertay
---

# Explainer: Intermediate Azure Active Directory

In the foundational adoption stage, you learned that Azure trusts Azure Active Directory (Azure AD) to provide authentication for users that want to create, read, update, or delete Azure resources. Let's move on to take a look at some other aspects of Azure AD that are important to keep in mind on your adoption journey.

First, Azure AD is a **multi-tenant** service. This is why a secure, dedicated instance of Azure AD is called a **tenant**. An Azure AD tenant is associated with at least one Azure administration account. It is possible for multiple Azure AD tenants to be associated with a single Azure administration account. This enables an organization to group users by tenant, but it does not allow organizations with their own Azure administrative accounts to share an Azure AD tenant.

## What is Azure AD used for?

* First decision: how many tenants
    * considerations:
        * will your users be aware that they are in Azure, or not? That is, your developers, IT, finance, etc. people will definitely know that they are in Azure; consumers of services implemented in Azure will not know that they are in Azure
        * more tenants equals more granularity, but increases management overhead (adding users, moving users, deleting users, associating service creds with users, etc.)
        * less tenants can mean more users per tenant, and you'll have to manage users with different features such as groups
        * reporting: reporting is per tenant; no one to see an all up report for multiple Azure AD tenants
        * do you want to use a tenant as a security boundary, to prevent users in one tenant from seeing resources in a second tenant; from 


(questions in no particular order right now)
* do you have an existing on-premises Active Directory Domain Services with user accounts that you want to use for identity in Azure?
    * yes - Azure AD connect
        * do you want to continue to store identity **only** on-premises?
            * yes - use federation
            * no - use synchronization
    * no - store all user accounts in Azure AD
* for your line of business workloads that will execute 
* do you need to integrate Azure first party and third party **service** authentication using Azure AD?
    * yes
    * no
* do you n 
