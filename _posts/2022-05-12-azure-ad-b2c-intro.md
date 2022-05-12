---
title: "Customer Identity And Access Management Using Azure AD B2C - Introduction"
excerpt: "In this post we are going to talk about how to build customer identity access management using Azure AD B2C."
header:
  overlay_image: /images/code.jpeg
date: 2022-05-12
tags: [Azure B2C, Azure, Identity, CIAM]
---

Customer identity and access management is one of the core features every business needs to get right when building a customer facing software product. Due to the numerous standards one has to adhere to, as well as constant maintenance required to ensure that customer information is well secured, I personally prefer not to build my own identity management solution. There are a number of identity as a service solutions out there and Azure AD B2C is one of them. In this post we are going to introduce Azure AD B2C and how it can assist you to manage your customer identity and access control. This is going to be the first post in a mini series where we will dive deeper into Azure AD B2C.

## What Is Azure AD B2C

Azure AD B2C is a highly scalable customer identity and access management (CIAM) solution provided by Microsoft that lets you build user journeys for your consumer and customer facing applications. It allows businesses to allow their customers to sign in to their products using various methods including social accounts, personal and/or work emails. In addition to that, it takes care of the safety of the authentication platform, monitoring, and automatically handling threats like denial-of-service, password spray, or brute force attacks. It uses standards-based authentication protocols including OpenID Connect, OAuth 2.0, and Security Assertion Markup Language (SAML) which makes it easy to integrate with modern applications.

Azure AD B2C provides a white labeled authentication solution. The entire user experience and be customized and branded to meet your needs. You can also extend the schema to store addition user attributes (up to 100 of them) tailor-made for your business. We will discuss how to use custom attributes in a later post. With translations for 36 languages (at the time of writing), Azure AD allows you to reach users around the globe without too much work required from you. _Figure 1_ below, which I uprooted from the documentation, gives an overview of how Azure AD B2C works.

<figure>
<img src="{{ site.baseurl }}/images/b2c.png" alt="B2C overview">
<figcaption>Figure 1: B2C overview. Credit: Azure AD B2C documentation (https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview)</figcaption>
</figure>

We are now going to talk about the some concepts you need to know when working with Azure AD B2C.

## Azure AD B2C Concepts

At the heart of Azure AD B2C is a tenant. A tenant is a directory of users that represent your organization. An Azure AD B2C tenant should not be confused with your Azure AD tenant. In fact, Azure AD B2C is not the same as Azure AD, even though it is built with the same technology. You may have up to 20 Azure AD B2C tenants in one subscription and each one is distinct and separate from the others. Creating a tenant is the first task you need to complete before you start working with Azure AD B2C. You can visit the [documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant) to find out how to do that. A Azure AD B2C tenant contains these resources, among others:

- **Directory** - This is where Azure AD B2C stores user credentials, profiles and app registrations.
- **App registrations** - This is where you register your applications and APIs that you want to protect
- **Identity experiences** - Azure AD B2C provides two options to ways of creating user journeys for your customers -- user flows and custom policies. User flows provide a graphical interface to quickly set up tasks like sign in, sign up and profile editing. If you don't have complex requirements for your identity experience then user flows will suffice. Custom policies give you more control over how you set up your identity tasks and are suitable for situations where you need to create complex identity workflows. In this series we will focus on custom policies since we are going to try and solve a fairly complex case study.

## Conclusion

In this post we introduced Azure AD B2C, a scalable customer identity and access management (CIAM) solution provided by Microsoft. This was more or less a summary of the extensive [documentation](https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview) that I highly recommend you read and understand if you want to start working with Azure AD B2C. In the next post we are going to create an Azure AD B2C tenant and set up a custom policy to get up and running. Thank you so much for taking your time to read and feel free to leave a comment, suggestion or question.

## Further reading

- [Azure AD B2C overview](https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview)
- [Azure AD B2C technical overview](https://docs.microsoft.com/en-us/azure/active-directory-b2c/technical-overview)
- [Azure AD B2C supported features](https://docs.microsoft.com/en-us/azure/active-directory-b2c/supported-azure-ad-features) - this will help you understand the differences between Azure AD and Azure AD B2C in terms of supported features.
- You may check out this [link](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/external-identities-overview) if you want an overview to Azure AD external identities and how Azure AD B2C fits in.
