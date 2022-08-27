---
title: "Introduction To Azure Managed Identities"
excerpt: "In this post we are going to introduce Azure managed identities"
header:
  overlay_image: /images/code.jpeg
date: 2022-08-27
tags: [Azure, Managed Identity]
---

Almost every software project has some kind of secret that is required for the product to run. Examples include database connection strings, access/API keys to third party services and certificates to mention a few. These secrets need to be handled with care because if they fall into wrong hands chaos will definitely ensue. Just imagine a hacker getting their hands on your database connection string! Wouldn't it be better if developers don't have to handle secrets at all? Thanks to managed identities, that is possible in Azure.

## What Is A Managed Identity

Before we define what a managed identity is, let's start by talking about what Azure Active Directory (Azure AD) is. Azure AD is a cloud-based identity and access management service which enables organisations to manage identities and control access to resources. In order for an entity to access resources that are protected by Azure AD, it must be represented by a security principal. The security principal is given a set of permissions which govern what the entity can and cannot do.

There are two main types of security principals -- a user principal and a service principal. A user principal is a security principal that represents a human being while a service principal is a security principal that represents a service or application. Instead of having to create fake user accounts to authenticate your daemon applications for instance, you can use a service principal instead.

A managed identity is a service principal that is automatically created and managed for you by Azure. Like any other security principal, you can use role-based access control (RBAC) to manage what your managed identity is and is not allowed to do. Unlike a normal service principal that can be used by anything as long as it has the credentials, a managed identity can only be linked to Azure resources. Since managed identities are created and managed by Azure, it helps eliminate some of the drawbacks that service principals have. Here are some of the drawbacks:

- Someone has to manually create a service principal. With managed identities the service principal is automatically created for you.
- The service principal's credentials are known to the person who created it as well as the entity that's using it. This is a security risk. With managed identities the credentials are not exposed in any way.
- One of the major drawbacks of service principals that might catch you out if you are not careful is that its credentials are valid for 1 year by default for security reasons. This means that someone needs to be aware of this and generate a new credential before it expires, then update the credential in all the consumers of the service principal. This sounds like a tedious process. Managed identities manage all this for you so you never have to worry about expiring credentials.

Managed identities totally eliminate the need for developers to manage secrets. I for one am not good at keeping secrets so this comes in handy :wink:.

## System-Assigned vs User-Assigned Manage Identities

There are two types of managed identities -- system-assigned and user-assigned. System-assigned managed identities are created as part of a resource. Their lifecycles are linked to the resource's lifecycle so if you delete the resource the managed identity will also be deleted. A system-managed identity can only be assigned to one resource.

User-assigned identities on the other hand are created by a user as stand-alone resources in Azure. They are then attached to one or many resources and their lifecycles are independent from the resources' lifecycles. For more details on the difference between the two please check the [documentation](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types).

### Which One Should I Use?

After seeing the differences between a system-assigned and a user-assigned managed identity the next question to ask is which type should one use. The answer, like all answers in software development, is that it depends. However, user-assigned managed identities have a lot of advantages compared to system-assigned managed identities. One of their biggest advantages is that one user-assigned managed identity can be assigned to more than one resource thereby reducing administrative overhead. For a more comprehensive view on the pros and cons of both types you can check out the [documentation](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identity-best-practice-recommendations#choosing-system-or-user-assigned-managed-identities).

**Figure 1** below shows a visual representation of different types of security principals and where managed identities fit in.

<figure>
<img src="{{ site.baseurl }}/images/managed-identities.svg" alt="Security principals">
<figcaption>Figure 1: Visual representation of security principals in Azure</figcaption>
</figure>

## How Can You Use Managed Identities

Like we said earlier, managed identities can help take away the burden of secrets management from developers. But how can one achieve this? You can keep all your secrets in [Azure key vault](https://azure.microsoft.com/en-us/services/key-vault/#getting-started). You then create a managed identity and give it read permissions to the key vault and assign it to the resource that needs to use the secrets. The consuming resource will then use the managed identity to authenticate to the key vault and fetch the secrets it needs.

> Whenever you are granting permissions to a managed identity, or any security principal, remember to use the [principle of least privilege](https://www.f5.com/labs/articles/education/what-is-the-principle-of-least-privilege-and-why-is-it-important).

If using a key vault is an overkill for you and all you want is to access one resource, say Cosmos DB, you can grant your managed identity permissions to the Cosmos DB and you are good to go. The example we will use in the next post uses this approach.

## What's Next?

Now that we have introduced Azure managed identities, in the next post we are going to create a simple web API that uses a managed identity to connect to Table Storage. Thanks for taking your time to read and please don't hesitate to leave a comment, question or suggestion.
