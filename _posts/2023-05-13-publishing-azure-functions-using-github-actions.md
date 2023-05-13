---
title: "Azure Functions 101: Publishing Azure Functions Using GitHub Actions"
excerpt: "In this post I will be showing you how to publish Azure Functions using GitHub Actions."
date: 2023-05-13
tags: [Azure, Azure Functions, Serverless]
---

This is the fifth post in a series: [Azure Functions 101]({{ site.baseurl}}/series/azure-functions-101). In this series, I will be covering the following topics:

- Part 1: [What Is Azure Functions]({{ site.baseurl}}/azure-functions-intro)
- Part 2: [Anatomy of Azure Functions]({{ site.baseurl}}/azure-functions-anatomy)
- Part 3: [Deploying Azure Function Resources Using Bicep and GitHub Actions]({{ site.baseurl}}/deploy-azure-function-resources)
- Part 4: [Creating Azure Functions using the Azure Functions Core Tools]({{ site.baseurl}}/creating-azure-functions-using-core-tools)
- Part 5: Publishing Azure Functions Using GitHub Actions (this post)

In the previous post we created our Todo API using the Azure Functions Core Tools and ran it locally. Now we need to deploy it to Azure. In this post I will be showing you how to publish Azure Functions using GitHub Actions. This is going to be a very short post. Let's get started.

## Get the Publish Profile

The first thing we need to do is get the publish profile for our Azure Function. To do this, navigate to your Azure Function in the Azure Portal -- the one you deployed in [part 3]({{ site.baseurl}}/deploy-azure-function-resources) if you were following along this series. Click on the **Get publish profile** button. This will download a file containing the publish profile for your Azure Function. Copy the contents of this file and add them to a GitHub secret called `AZURE_FUNCTIONAPP_PUBLISH_PROFILE`. You can read more about GitHub secrets [here](https://docs.github.com/en/actions/reference/encrypted-secrets).

> I really would have loved to get the publishing profile in the Bicep template and add it to the GitHub workflow as an output to the 'publish' job. Unfortunately, after numerous attempts, I couldn't get it working. If you know how to do this, please let me know in the comments section below. I will greatly appreciate it.

## Update the GitHub Workflow

Now we need to update the GitHub workflow we created in [part 3]({{ site.baseurl}}/deploy-azure-function-resources) to publish our Azure Function. Update it so it looks like this:

![GitHub workflow](/images/azure-functions/publishing-azure-functions.svg)

We have added another job to the workflow called `publish-function`. This job will run after the `deploy-infrastructure` job since it needs to wait for the resources to be successfully deployed before it can publish the Azure Function. The `publish-function` jobs gets the function app name from the `deploy-infrastructure` job -- the output we declared in our Bicep template. If we push our code to GitHub now, the workflow will run and publish our Azure Function.

## Conclusion

That's it. Now when you push your code to GitHub, the GitHub workflow will run and publish your Azure Function. You can find the code for this series on [GitHub](https://github.com/vince-nyanga/azure-functions-demo). This is the last post in this series. I hope you enjoyed it. If you have any questions or comments, please leave them in the comments section below.
