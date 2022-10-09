---
title: "Using Azure Managed Identities"
excerpt: "In this post we are going to talk about how to use Azure managed identities."
header:
  overlay_image: /images/code.jpeg
date: 2022-10-09
tags: [Azure, Managed Identity]
---

In the [previous post]({{ site.baseurl}}/managed-identity) we introduced Azure Managed Identities -- a secure way to access Azure resources without the need to manage secrets. In this post we are going to talk about how to use managed identities in your code. It is going to be a relatively short read.

## Getting Started

To use Azure managed identities you need to install the `Azure.Identity` package into your project. Most azure resources now have constructors that take in the resource URI and a [`TokenCredential`](https://learn.microsoft.com/en-us/dotnet/api/azure.core.tokencredential?view=azure-dotnet) which is a abstract class that represents a credential capable of providing an OAuth token. There are a lot of classes derived from `TokenCredential`, `ManagedIdentityCredential` being one of them. We are however not going to use `ManagedIdentityCredential` as there is a more versatile class -- [`DefaultAzureCredential`](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet).

## Using `DefaultAzureCredential`

The `DefaultAzureCredential` class provides a default `TokenCredential` authentication flow for applications that will be deployed to Azure. It tries to find a credential from various sources in the hierarchy shown in **Figure 1** below.

<figure>
<img src="{{ site.baseurl }}/images/defaultcredential.svg" alt="Default Azure Credential hierarchy" style="height:400px;">
<figcaption>Figure 1: The hierarchy in which DefaultAzureCredential looks for credentials.</figcaption>
</figure>

Now in your code you can use `DefaultAzureCredential`. When you are running your code on your local machine the `DefaultAzureCredential` will try to get your credential from various sources such as Visual Studio, Visual Studio Code or Azure CLI. Once you deploy your code to Azure, managed identities will be used. If you want to use a user-assigned managed identity you need to provide the identity's client ID (see code snippet below). If no client ID is provided, the system-assigned managed identity will be used.

Here is an example of how you can connect to Azure Table storage using managed identities. The code from which the snippet was taken can be found on [GitHub](https://github.com/vince-nyanga/AzureManagedIdentity) if you would like to take a look.

```csharp
public static void AddTableStorage(this IServiceCollection services, IConfiguration configuration)
{
    var tableStorageUri = configuration["TableStorageUri"];

    // User-assigned managed identity
    var managedIdentityClientId = configuration["IdentityClientId"];

    // If you are using a system-assigned managed identity you don't need the options
    var options = new DefaultAzureCredentialOptions
    {
        ManagedIdentityClientId = managedIdentityClientId
    };

    if (Uri.TryCreate(tableStorageUri, UriKind.Absolute, out var tableUri))
    {
        var tableServiceClient = new TableServiceClient(tableUri, new DefaultAzureCredential(options));
        services.AddSingleton<ITableStorageService>(new TableStorageService(tableServiceClient));
        services.AddTransient<ITableStorageRepository, TableStorageRepository>();
    }
    else
    {
          throw new Exception("Invalid table storage URI");
    }
}
```

## Conclusion

In this post we spoke about how you can use Azure managed identities in your code using the `DefaultAzureCredential` class. Like always, thanks so much for taking your time to read and don't hesitate to leave a question or contribution in the comments section below.
