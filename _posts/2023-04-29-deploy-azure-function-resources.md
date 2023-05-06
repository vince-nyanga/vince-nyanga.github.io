---
title: "Azure Functions 101: Deploying Azure Function Resources Using Bicep and GitHub Actions"
excerpt: "In this post I will be showing you how to deploy Azure Function resources using Bicep and GitHub Actions."
date: 2023-04-29
tags: [Azure, Azure Functions, Serverless]
---

This is the third post in a series: [Azure Functions 101]({{ site.baseurl}}/series/azure-functions-101). In this series, I will be covering the following topics:

- Part 1: [What Is Azure Functions]({{ site.baseurl}}/azure-functions-intro)
- Part 2: [Anatomy of Azure Functions]({{ site.baseurl}}/azure-functions-anatomy)
- Part 3: Deploying Azure Function Resources Using Bicep and GitHub Actions (this post)
- Part 4: Creating Azure Functions using the Azure Functions Core Tools (coming soon)
- Part 5: Publishing Azure Functions Using GitHub Actions (coming soon)

In this post, I will show you how to deploy Azure Function resources using [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep) and [GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions). We will deploy the following Azure resources:

- App Service Plan
- Storage Account
- Azure Function App

**Table of Contents**

- [Prerequisites](#prerequisites)
- [Creating the Bicep template](#creating-the-bicep-template)
- [Creating a service principal](#creating-a-service-principal)
- [Deploying the Bicep template using GitHub Actions](#deploying-the-bicep-template-using-github-actions)
- [Conclusion](#conclusion)

## Prerequisites

If you want to follow along with this post, you will need to have an [Azure account](https://azure.microsoft.com/en-us/free/) as well as a [GitHub account](https://github.com/). You should also have the Bicep extension installed in Visual Studio Code (I used VS Code but you are free to use any editor you like). You can install the Bicep extension by following the instructions [here](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-bicep). I'm making an assumption that you are familiar with the basics of Bicep and GitHub Actions. If you are not, I recommend that you read the following articles:

- [Bicep overview](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview)
- [GitHub Actions overview](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)

## Creating the Bicep template

We are going to create a Bicep template that will deploy the resources mentioned above. We will create a new file called `function-app.bicep` in the `infrastructure` directory.

![Bicep template](/images/az-func-bicep.svg)

Now that we have created the Bicep template, we can create a parameters file called `function-app.parameters.json` in the `infrastructure` directory:

![Bicep parameters](/images/az-func-params.svg)

Since all of the parameters have default values, I only need to specify the `storageAccountSkuName` parameter in the parameter file to show how you can specify your own parameter values. You can deploy the Bicep template above without specifying any parameters.

## Creating a service principal

In order to deploy the Bicep template using GitHub Actions, we need to create a service principal. We will create a new service principal called `bicep-deployer` and assign it the `Contributor` role on the resource group that we will be deploying the resources to:

![Service principal](/images/az-func-service-principal.svg)

We don't want to assign the `Contributor` role at the subscription level because that would give the service principal access to all the resources in the subscription which is not desirable. We are using the principle of least privilege here.

## Deploying the Bicep template using GitHub Actions

We need to create a new secret in the GitHub repository called `AZURE_CREDENTIALS` and set the value to the JSON output from the command above. We also need to create two more secrets called `AZURE_SUBSCRIPTION` and `AZURE_RG` and set the values to the subscription ID and resource group name respectively. The secrets will be used in the GitHub Actions workflow.

We will create a new workflow file called `main.yml` in the `.github/workflows` directory:

![GitHub Actions](/images/az-func-github-action.svg)

The workflow will run when a push is made to the `main` branch or when the workflow is manually triggered. The workflow will use the `azure/login` action to login to Azure using the service principal credentials we stored in our secrets. It will then use the `azure/arm-deploy` action to deploy the Bicep template.

## Conclusion

In this post, we have created a Bicep template that deploys an Azure Function App and a Storage Account. We have also created a GitHub Actions workflow that deploys the Bicep template to Azure. I hope you found this post useful. If you have any questions or comments, please leave them below. In the next post, we will look at how to create and run Azure functions using the Azure Functions Core Tools.
