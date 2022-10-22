---
title: "Using GitHub Actions To Deploy Angular Website To Azure Blob Storage"
excerpt: "In this post we are going to talk about how to use GitHub actions to deploy an Angular website to Azure blob storage."
header:
  overlay_image: /images/code.jpeg
date: 2022-10-22
tags: [Azure, GitHub Actions, CI/CD]
---

In this very short post I am going to talk about how to deploy an Angular website to an Azure storage account using GitHub actions. A few weeks ago I needed to create a CI/CD pipeline for my personal project. This post is a compilation of what I did to achieve that to save you time in case you have a similar problem.

I am going to assume that you are already familiar with how to host a website in an Azure storage account. If you don't know how, please check out this [link](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website). Let's get started.

## The Service Principal

We need to create an Azure AD service principal that we are going to give permissions to write to our storage account. We are going to use the Azure CLI to achieve that:

```bash
resourceGroup="[your-resource-group]"

subscriptionId="[your-subscription-id]"

az ad sp create-for-rbac \
    --name revouchers \
    --role contributor \
    --scopes /subscriptions/$subscriptionId/resourceGroups/$resourceGroup \
    --sdk-auth
```

The above script creates a service principal that has a `Contributor` role to the resource group that contains the storage account. The `Contributor` role is quite powerful so you might need to use the principle of least privilege which I did not use in this instance :smile:.

The output of this command will be the credentials of the service principal. You need to keep these safe! Now copy these credentials and add them to your GitHub repository secrets. I named my secret `AZURE_CREDENTIALS` but you can call it whatever you want. Now let's turn our attention to the Angular project.

## Update package.json

Go to the `package.json` file in your Angular project and add the following to the `scripts` section:

```json
{
  //...
  "scripts": {
    // ...
    "build:ci": "ng build --configuration production",
    "test:ci": "ng test  --watch=false --code-coverage --browsers ChromeHeadless"
  }
  // ...
}
```

The two scripts above will help us build and test our Angular website when the GitHub action workflow runs. Let's now create our GitHub action workflow.

## The GitHub Action Workflow

I'm assuming that you have knowledge of GitHub actions. For those who might not know what GitHub actions is, it is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. Check out this [link](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) to find out more. The workflow is going to do the following:

- Build the Angular website
- Run tests
- Upload build artifacts to a storage account hosting the website
- Purge the CDN endpoint connected to the storage account to ensure users get the latest changes. This is an optional step if you don't have a CDN in front of your website. I highly recommend it though and it won't cost you that much. To see how much it costs me for a similar set up (with a lot more going on), check out [this post] ({{ site.baseurl}}/saving-money-with-azure).

```yaml
name: Deploy Website

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  PROJECT_NAME: [your-angular-project-name]
  RESOURCE_GROUP: [your-resource-group-name]
  STORAGE_ACCOUNT_NAME: [your-storage-account-name]
  CDN_PROFILE_NAME: [your-cdn-profile-name]
  CDN_ENDPOINT_NAME: [your-cdn-endpoint-name]

jobs:
  build:
    name: Build and test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Install GitVersion
        uses: gittools/actions/gitversion/setup@v0.9.7
        with:
          versionSpec: "5.x"
      - name: Determine Version
        id: gitversion
        uses: gittools/actions/gitversion/execute@v0.9.7
        with:
          useConfigFile: true
      - name: Update version in package.json
        run: |
          sed -i "s/0.0.1/$GITVERSION_SEMVER/g" package.json
      - name: Use Node 16.x
        uses: actions/setup-node@v1
        with:
          node-version: "16.x"
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build:ci
      - name: Test
        run: npm run test:ci
      - name: Archive build
        if: success()
        uses: actions/upload-artifact@v1
        with:
          name: deploy_dist
          path: dist
      - name: Archive code coverage result
        if: success()
        uses: actions/upload-artifact@v1
        with:
          name: deploy_coverage
          path: coverage
  deploy:
    name: Deploy to Azure
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout
        uses: actions/checkout@v1
      - uses: azure/login@v1
        with:
          creds: {% raw %} ${{ secrets.AZURE_CREDENTIALS }} {% endraw %}
      - name: Download build
        uses: actions/download-artifact@v1
        with:
          name: deploy_dist
      - name: Upload to blob storage
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az storage blob upload-batch --account-name $STORAGE_ACCOUNT_NAME --auth-mode key -d '$web' -s ./deploy_dist/$PROJECT_NAME --overwrite
      - name: Purge CDN endpoint
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az cdn endpoint purge --content-paths  "/*" --profile-name "$CDN_PROFILE_NAME" --name "$CDN_ENDPOINT_NAME" --resource-group "$RESOURCE_GROUP"
      - name: logout
        run: |
          az logout
        if: always()
```

The names in the workflow steps are descriptive enough. I don't think there is any need to explain what's going on.

That's it. This is how you can use GitHub actions to deploy an Angular website hosted in a storage account on Azure. Like always, thank you so much for taking your time to read. If you have any question, comment or suggestion, add it to the comments section below.
