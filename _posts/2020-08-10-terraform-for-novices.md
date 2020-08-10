---
title: "Terraform For Novices Like Me - Introduction"
date: 2020-08-10
tags: [Terraform, Infrastructure As Code]
header:
  overlay_image: /images/code.webp
  overlay_filter: 0.5
  caption: "Photo credit: [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)"
excerpt: "An introduction to Terraform"
---

Until just a few months ago I had never heard of Terraform, let alone the term `Infrastructure As Code` (I'm a bit ashamed but it is what it is). That, however, is not the case anymore as I now at least know these two and have a rough idea of what they are. For the past few weeks I've been learning about infrastructure as code and Terraform in particular. I've been taking notes in the process and in this post I will share these notes with my fellow novices. If you are not a novice like me when it comes to Terraform this post is not for you I'm afraid. First, let's talk about infrastructure as code.

## What Is Infrastructure As Code

According to [this website](https://searchitoperations.techtarget.com/definition/Infrastructure-as-Code-IAC), infrastracture as code is a type of IT setup wherein developers or operations teams **automatically** manage and provision the technology stack for an application **through software**, rather than using a manual process to configure discrete hardware devices and operating systems. In simpler terms, instead of manually setting up your infrastructure, you write some code to do it for you. That's cool right? :sunglasses:

This has many advantages that include predictability -- there is no room for human error anymore so you know exactly what you will get. You can also reuse your code, version, debug it and revert to previous versions if something goes wrong like you would do with any code you write. Now let's look at Terraform.

## Terraform

Terraform is a tool for building and managing infrastructure by providing declarative execution plans for building and running applications and infrastructure. With declarative plans, you specify the outcome that you want and Terraform will take care of the process. You just tell it 'I want a SignalR service with this configuration' and it will give you exactly that without you having to know every step it takes to get there. One of Terraform's greatest advantages is that it works with a lot of vendors including AWS, Google Cloud Platform and Azure.

In this post I'm going to talk about providers and resources, which are the building blocks for your code. I will also share a simple example where we provision a SignalR service on Azure using Terraform. In the next post I'll talk about variables and modules while expanding the SignalR example. The reason why I break down these posts is that I personally don't enjoy reading posts so I try to make my posts as short as possible and as a result, I'm not going to talk about how to install Terraform on your machine.

### Providers

In Terraform, providers are what I call the backbone of the whole operation. They connect terraform to the infrastructure you want to manage for instance, AWS or Azure. They provide all the configuration like connection details and authentication/authorization details. This is how you declare a provider in Terraform's HashiCorp Configuration Language:

```hcl
provider "[PROVIDER_NAME]" {
    # your configuration here
}
```

In our example we will be using the Azure Resource Manager provider so our code will look like this:

```hcl
provider "azurerm" {
  # it is recommended to lock the version
  version = "=2.20.0"
  features {}
}
```

For more details on how to use the Azure provider visit the [Terraform documentation](https://www.terraform.io/docs/providers/azurerm/index.html). Now that we know what a provider is let's take a look at resources.

### Resources

Resources are the infrastructure components that you want to manage such as databases, networks etcetera. In our case we want to provision a SignalR resource. The syntax for creating resources is as follows:

```hcl
resource "[PROVIDER_NAME]_[RESOURCE_TYPE]" "[NAME]" {
    # your configuration here
}
```

Note that the `[NAME]` section can be any name you like. In our SignalR example here's how we will do it:

```hcl
resource "azurerm_signalr_service" "signalr" {
  # ...
}
```

As you can see, the provider is `azurerm`, the resource type is `signalr_service` and the resource name is `signalr`. Let's now put everything together and provision some infrastructure.

### Example

Like I said earlier, we are going to use Terraform to provision a SignalR service for our fictitious application. Create a file called `signalr.tf` and let's get started. First we declare our provider:

```hcl
 provider "azurerm" {
  version = "=2.20.0"
  features {}
}
```

I'm not providing authentication details in this case because I have logged in to the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on my local machine so Terraform will get everything it needs from there.

Next we create our resource group. This is a Terraform resource specific to Azure.

```hcl
resource "azurerm_resource_group" "group" {
  name     = "hones-rg"
  location = "North Europe"
}
```

Now that we have our resource group, let's create our SignalR service:

```hcl
resource "azurerm_signalr_service" "signalr" {
  name                = "my-signalr"
  location            = azurerm_resource_group.group.location
  resource_group_name = azurerm_resource_group.group.name

  sku {
    name     = "Free_F1"
    capacity = 1
  }

  features {
    flag  = "ServiceMode"
    value = "Serverless"
  }
}
```

Do you see how we are referencing the resource group in our configuration for the SignalR service? We use the following syntax:

```
[RESOURCE_TYPE].[NAME].[VARIABLE_WE_WANT]
```

For instance, when we want to access our resource group name we use:

```
azurerm_resource_group.group.name
```

This is how your `signalr.tf` should look like:

```hcl
# Provider
provider "azurerm" {
  version = "=2.20.0"
  features {}
}

# Resource group
resource "azurerm_resource_group" "group" {
  name     = "hones-rg"  # use any name
  location = "North Europe" # the region you want your resource group to reside
}

# SignalR service
resource "azurerm_signalr_service" "signalr" {
  name                = "my-signalr"
  location            = azurerm_resource_group.group.location
  resource_group_name = azurerm_resource_group.group.name

  sku {
    name     = "Free_F1"
    capacity = 1
  }

  features {
    flag  = "ServiceMode"
    value = "Serverless"
  }
}
```

We now have everything in place so we can run the following commands:

1. `terraform init`: initialize everything and download required plugins
2. `terraform validate`: (optional) to validate that you got the syntax right
3. `terraform fmt`: (optional) to nicely format your `.tf` files
4. `terraform plan`: to create an execution plan
5. `terraform apply`: to apply the changes required to reach the desired state of the configuration. This operation will create the resource group and SignalR service on Azure.
6. `terraform destroy`: (if you want to) to destroy the Terraform-managed infrastructure. This command will delete your resource group and SignalR service.

## Conclusion

In this post I spoke about infrastructure as code (IaC) and the advantages it provides. I also introduced Terraform, a wonderful tool that helps provision and manage infrastructure on multiple vendors. I also spoke about providers and resources -- what I call the building blocks for terraform. Finally, I showed how one can use Terraform to provision a SignalR service on Microsoft Azure. Like I said earlier, this post is part of the notes I took while I was learning (I'm still learning) Terraform. Thank you so much for taking your time to read and I hope you have learnt something.

#### Further Reading And References

- [SearchITOperations](https://searchitoperations.techtarget.com/definition/Infrastructure-as-Code-IAC) website for the definition of infrastructure as code
- [Infrastructure as Code DevOps principle: meaning, benefits, use cases](https://medium.com/@FedakV/infrastructure-as-code-devops-principle-meaning-benefits-use-cases-a4461a1fef2)
- [Terraform website](https://www.terraform.io/intro/index.html)
- [The Terraform Book](https://terraformbook.com/)
- [Terraform - Getting Started](/courses/getting-started-terraform/table-of-contents) Pluralsight course by Ned Bellavance
