---
title: "Terraform For Novices Like Me - Modules, Input Variables And Output Values"
date: 2020-08-16
tags: [Terraform, Infrastructure As Code]
header:
  overlay_image: /images/code.webp
  overlay_filter: 0.5
  caption: "Photo credit: [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)"
excerpt: "An introduction to Terraform - modules, input variables and output values"
---

In the previous [post]({{ site.baseurl }}/terraform-for-novices/) I spoke about infrastructure as code (IaC) and how Terraform can be used to provision and manage infrastructure. I also gave a simple example where I provisioned a simple Azure SignalR service using Terraform. In this post I am going to talk about how to make use of modules, input variables and output values in Terraform and improve our example in the previous post. Let's get started.

## Modules

Every software engineer knows about the `DRY` principle (I hope), which stipulates that you shouldn't repeat yourself unless if it's absolutely necessary. This principle also applies to Terraform and modules help in achieving `DRY` code. In Terraform, modules are containers of resources that are used together and can be shared and reused across multiple configurations. Here's how you call a module in your configuration:

```hcl
module "[MODULE_NAME]" {
    source = "[link_to_module_source]"
}
```

## Input Variables

In order for modules to be shared and reused in multiple configurations they need to be configurable and customizable. For example, you may need to use the same module for your development environment and production environment but with slightly different setups. This is where input variables come in. They serve as parameters for a module, allowing the module to be customized without altering the module's source code. Here's the syntax for declaring input variables:

```hcl
variable "[VARIABLE_NAME]"{
    type        = [VARIABLE_TYPE]
    description = "[DESCRIPTION]"
    default     = [DEFAULT_VALUE]
}
```

Accessing input variables in your configuration is very easy:

```hcl
value = var.[VARIABLE_NAME]
```

The are many ways to populate your variables in Terraform. These are:

- Load from command line flag:
  ```bash
  terraform plan -var 'variable=value'
  ```
- Load from file -- Create a `terraform.tfvars` file and add your values therein:
  ```hcl
  [variable_name] = value
  ```
- Load from environment variables:
  ```bash
  export TF_VAR_[variable_name]=value
  ```
- Use default values when you declare the variables

## Output Values

In most cases you may want to get values out of your modules after running `terraform apply`, for instance, database connection strings. This is made possible in Terraform by using output values. In simple terms, output values are the return values of a Terraform module and have several uses as shown in the [docs](https://www.terraform.io/docs/configuration/outputs.html). Here's the sytanx for declaring output values:

```hcl
output "[NAME]" {
    value = [RESOURCE.PROPERTY]
}
```

## Example

Let's improve on our example from the previous post and make use of modules, input variables and output values. All the source code can be found on [GitHub](https://github.com/vince-nyanga/terraform-example) in case you want to get the complete code. First create the following directory structure:

```
terraform-example/
  - modules/
      - signalr/
```

### SignalR Module

Go to the `signalr` directory and add the following files:

#### 1. `variables.tf`

This file will contain all the input variables we need in our simple module:

```hcl
variable "name" {
  description = "The SignalR service name"
  type        = string
}

variable "resource_group_name" {
  description = "The resource group to which the SignalR service belongs"
  type        = string
}

variable "location" {
  description = "The location of the SignalR service"
  type        = string
}
```

#### 2. `outputs.tf`

This file will contain all the output values that our module will return:

```hcl
output "signalr_primary_key" {
  value = "${azurerm_signalr_service.this.primary_access_key}"
}
output "signalr_secondary_key" {
  value = "${azurerm_signalr_service.this.secondary_access_key}"
}
output "signalr_primary_connection_string" {
  value = "${azurerm_signalr_service.this.primary_connection_string}"
}
output "signalr_secondary_connection_string" {
  value = "${azurerm_signalr_service.this.secondary_connection_string}"
}
```

#### 3. `main.tf`

This is where we declare all the resources that our module will contain. In this case we only have one resource -- a SignalR service:

```hcl
resource "azurerm_signalr_service" "this" {
  name                = var.name
  location            = var.location
  resource_group_name = var.resource_group_name

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

### Configuration

We are done with our module. Now let's make use of the module in our configuration. Go to the root directory and add the following files:

#### 1. `signalr-dev.tf`

```hcl
provider "azurerm" {
  version = "=2.20.0"
  features {}
}

resource "azurerm_resource_group" "this" {
  name     = "hones-rg-dev"
  location = "North Europe"
}

module "signalr" {
  source                = "./modules/signalr"
  name                  = var.signalr_name
  resource_group_name   = azurerm_resource_group.this.name
  location              = azurerm_resource_group.this.location
}
```

#### 2. outputs.tf

```hcl
output "primary_key" {
  value = "${module.signalr.signalr_primary_key}"
}

output "primary_connection_string" {
  value = "${module.signalr.signalr_primary_connection_string}"
}
```

We have all we need now. Let's run the `init` command to initialize terraform and download all the required plugins:

```bash
terraform init
```

When all is initialized, run the following command to check if all is valid:

```bash
terraform validate
```

If everything checks out then you can run `plan` command:

```bash
terraform plan
```

and the `apply` command:

```bash
terraform apply
```

I am assuming that you have the Azure CLI installed on your machine and you're already authenticated otherwise you won't be able to `apply` your configuration.

## Conclusion

In this post I spoke about how to use modules, input variables and output values in Terraform. In addition, I improved on the example from the previous post to use modules, input variables and output values. The source code can be found on [GitHub](https://github.com/vince-nyanga/terraform-example). Please note that this post is part of the notes I gathered while I was learning about Terraform and is by no means a comprehensive guide. I'm also learning as I write this :smile:. Like always, thanks so much for taking your time to read and hopefully you have learnt something. Keep well and stay safe.

#### Further Reading

- [Terraform modules docs](https://www.terraform.io/docs/configuration/modules.html)
- [Terraform input variables docs](https://www.terraform.io/docs/configuration/variables.html)
- [Terraform output values docs](https://www.terraform.io/docs/configuration/outputs.html)
