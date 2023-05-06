---
title: "Azure Functions 101: Creating Azure Functions Using the Azure Functions Core Tools"
excerpt: "In this post I will be showing you how to create Azure Functions using the Azure Functions Core Tools."
date: 2023-05-06
tags: [Azure, Azure Functions, Serverless]
---

This is the fourth post in a series: [Azure Functions 101]({{ site.baseurl}}/series/azure-functions-101). In this series, I will be covering the following topics:

- Part 1: [What Is Azure Functions]({{ site.baseurl}}/azure-functions-intro)
- Part 2: [Anatomy of Azure Functions]({{ site.baseurl}}/azure-functions-anatomy)
- Part 3: [Deploying Azure Function Resources Using Bicep and GitHub Actions]({{ site.baseurl}}/deploy-azure-function-resources)
- Part 4: Creating Azure Functions using the Azure Functions Core Tools (this post)
- Part 5: Publishing Azure Functions Using GitHub Actions (coming soon)

In this post I am going to show you how you can create and run Azure Functions using the [Azure Function Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local). I am using the Azure Functions Core Tools version 4.0.4736 on a Mac. You can find the instructions to download the Azure Functions Core Tools on your specific OS [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local). Let's get started!

**Table of Contents**

- [Initializing the Azure Function](#initializing-the-azure-function)
- [Models and Entities](#models-and-entities)
- [Creating the functions](#creating-the-functions)
- [Running the function](#running-the-function)
- [Conclusion](#conclusion)

## Initializing the Azure Function

We are going to create a Todo API using Azure functions with the following endpoints:

- `GET /api/todos`: Gets all todos
- `GET /api/todos/{id}`: Gets a todo by id

First, we need to create a new Azure Function project. To do this, open a terminal and run the following command:

```bash
func init TodoApi --dotnet
```

The `--dotnet` flag instructs the Azure Functions Core Tools to create a .NET project. You can also create a JavaScript project by using the `--javascript` flag. The command will create a new folder called `TodoApi` and initialize a new .NET project in that directory. The project will contain a `host.json` file and a `local.settings.json` file. The `host.json` file contains the configuration for the Azure Functions host. The `local.settings.json` file contains the configuration for the Azure Functions Core Tools. The `local.settings.json` file is used when running the Azure Functions locally. The `local.settings.json` file should not be committed to source control.

## Models and Entities

Next, we need to define our `Todo` model:

```csharp
namespace TodoApi.Models
{
    public record Todo(string Id, string TaskDescription, bool IsComplete);
}
```

Since we are going to store our todos in Azure table storage, we need to install the `Microsoft.Azure.WebJobs.Extensions.Tables` NuGet package. To do this, go into the `TodoApi` directory and run the following command:

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.Tables
```

We now need to create a class that implements `ITableEntity` which we will persist in our Azure table:

```csharp
namespace TodoApi.Entities
{
    public class TodoEntity : ITableEntity
    {
        public string PartitionKey { get; set; }
        public string RowKey { get; set; }
        public DateTimeOffset? Timestamp { get; set; }
        public ETag ETag { get; set; }
        public string TaskDescription { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

## Creating the functions

Next, we need to create a new Azure Function. Go into the `TodosApi` directory and run the following commands:

```bash
# POST /api/todos
func new --name CreateTodo --template "HTTP trigger"

# GET /api/todos/{id}
func new --name GetTodo --template "HTTP trigger"
```

These commands create two HTTP-triggered functions, one to create a `Todo` and the other one get a todo by ID. You should see two classes -- `CreateTodo.cs` and `GetTodo.cs`, both containing boilerplate code for an HTTP-triggered function. Now, we need to update the code inside the `CreateTodo.cs` class to look like this:

```csharp
namespace TodoApi
{
    public static class CreateTodo
    {
        [FunctionName("CreateTodo")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = "todos")] HttpRequest req,
            [Table(TableStorageConstants.TableName, Connection = "AzureWebJobsStorage")] IAsyncCollector<TodoEntity> todosCollector,
            ILogger log)
        {
            log.LogInformation("Creating a new todo.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

            TodoEntity todoEntity = CreateTodoEntity(requestBody);

            await todosCollector.AddAsync(todoEntity);

            log.LogInformation("Todo created successfully.");

            return new OkObjectResult(todoEntity.RowKey);
        }

        private static TodoEntity CreateTodoEntity(string requestBody)
        {
            var request = JsonConvert.DeserializeObject<CreateTodoRequest>(requestBody);

            return new TodoEntity
            {
                TaskDescription = request.Task,
                IsComplete = false,
                PartitionKey = TableStorageConstants.PartitionKey,
                RowKey = Guid.NewGuid().ToString()
            };
        }
    }
}
```

In addition to the trigger, the function has an output binding to an Azure table storage table. Notice how we just provide the table name as well as the configuration key to the connection string. This is one of the cool features of bindings. We don't need to worry about creating the `TableClient`. It does that for us :smile:.

Next, let's update the code in the `GetTodo.cs` class to look like this:

```csharp
namespace TodoApi
{
    public static class GetTodo
    {
        [FunctionName("GetTodo")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = "todos/{id}")] HttpRequest req,
            [Table(TableStorageConstants.TableName, TableStorageConstants.PartitionKey, "{id}", Connection = "AzureWebJobsStorage")] TodoEntity todoEntity,
            ILogger log)
        {
            log.LogInformation("Getting todo by id.");

            if (todoEntity == null)
            {
                return new NotFoundResult();
            }

            return new OkObjectResult(todoEntity.ToModel());
        }
    }
}
```

This function uses an input binding to get the entity from a table. It uses the `id` from the route to get the entity. This is another example of how useful bindings are.

## Running the function

To run the function locally, we need to ensure we have a valid connection string to the Azure table storage. I am running an Azure storage emulator locally using [Azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite). Once the function has been deployed, it will persist the entity in the storage account we deployed in the previous post. To run the function locally, go into the `TodoApi` directory and run the following command:

```bash
func start
```

## Conclusion

In this post I showed how you can use the Azure Functions Core tools to create an Azure function. You can look at the complete source code on [GitHub](https://github.com/vince-nyanga/azure-functions-demo). This is not the only way to create Azure functions. You can also use Visual Studio, Visual Studio Code, or the Azure Portal. I hope you found this useful. Please don't hesitate to leave a comment, question or suggestion in the comment section below. Once again, thanks for reading.
