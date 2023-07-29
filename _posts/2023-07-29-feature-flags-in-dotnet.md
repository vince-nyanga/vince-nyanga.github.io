---
title: "Using Feature Flags in .NET"
excerpt: "In this post, I will briefly explain how to use feature flags in .NET."
date: 2023-07-29
tags: [Feature Flags, .NET, C#]
---

Feature flags, also known as feature toggles, are a powerful tool in software development that allows developers to enable or disable specific features or code paths within an application. They provide flexibility and control over feature releases, enabling teams to deliver new functionality to users rapidly and safely, without needing to redeploy their applications.

In this comprehensive guide, we will explore the concept of feature flags in the .NET ecosystem. We will start with a basic understanding of feature flags and gradually delve into to implement feature flags in .NET. By the end of this article, you will have a solid grasp of how to effectively use feature flags in your .NET applications.

## Why feature flags?

Before we dive into the implementation details, let's take a moment to understand why feature flags are so valuable in the software development process. Feature flags offer several benefits that make them an essential tool for developers:

1.  **Gradual Feature Rollouts**: Feature flags allow for gradual feature rollouts, enabling teams to release new functionality to a subset of users or in specific environments. This helps mitigate risks and allows for early feedback and testing.
2.  **A/B Testing**: With feature flags, you can easily conduct A/B testing by enabling a feature for a specific group of users and comparing their behavior with a control group. This data-driven approach helps validate ideas and make informed decisions based on user feedback.
3.  **Hot Fixes and Rollbacks**: Feature flags provide the ability to quickly disable or roll back features in case of issues or bugs. This ensures that the impact on users is minimized and allows for rapid response to critical issues.
4.  **Trunk-Based Development**: Feature flags are particularly useful in trunk-based development, where all developers work on a single branch. By using feature flags, developers can isolate their changes and prevent destabilizing the codebase, avoiding the complexities of long-lived branches.
5.  **Continuous Delivery**: Feature flags promote continuous delivery by decoupling feature development from the feature release. This allows teams to continuously integrate and test features without disrupting the main codebase.

If you set up your feature flags using services like Azure App Configuration, you will be able to toggle your features without needing to redeploy your application.

## Setting up feature management in .NET

To start using feature flags in your .NET applications, you need to set up the feature management libraries. These libraries provide idiomatic support for implementing feature flags in .NET, including support for declarative feature flag configuration and lifecycle management. The first step is to add the `Microsoft.FeatureManagement.AspNetCore` NuGet package to your project. This package provides the necessary dependencies for feature flag management in .NET. Once you have added the package, you can configure the feature manager in your `Startup.cs` file. The feature manager retrieves feature flag configurations from the .NET Core configuration system, allowing you to define feature flag settings using various configuration sources such as `appsettings.json`, environment variables, or other services such as Azure app configuration.

To configure the feature manager, you need to call the `AddFeatureManagement` extension method on the `IServiceCollection`.

```cs
builder.Services.AddFeatureManagement();
```

This method sets up the feature manager with the default configuration location, which is the `"FeatureManagement"` section of the .NET Core configuration data. Your features will be declared in the `appsettings.json` file like this:

```json
{
  "FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
      // percentage-base rollout
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": {
            "Value": 50
          }
        }
      ]
    }
  }
}
```

With this basic setup, you can now start using feature flags in your .NET application. In the next section, we will explore how to add feature flags to different parts of your application to control feature availability.

## Adding feature flags to your application

Feature flags can be added to various parts of your .NET applications be it web APIs or web apps, to control the availability of specific features. Let's explore different areas where you can add feature flags in your .NET application:

### 1. Controller actions

One common use case is to enable or disable specific controller actions based on feature flags. This allows you to control which actions are accessible to users based on their feature flag configuration. To add feature flags to a controller action, you can use the `[FeatureGate]` attribute. This attribute allows you to specify the feature flag that should be checked before executing the action. Here's an example of how to add a feature flag to a controller action:

```cs
[HttpGet]
[FeatureGate("my-feature-flag")]
public IActionResult Get()
{
    // Your action logic
}
```

### 2. Views

Feature flags can also be used to control the visibility of specific views in your Razor pages. This is useful when you want to show or hide certain UI elements or sections based on feature availability.

```razor
@inject Microsoft.FeatureManagement.IFeatureManager featureManager

@if (await featureManager.IsEnabledAsync("my-feature-flag"))
{
    <div>
    This content is only visible when the 'my-feature-flag' feature flag is
    enabled.
    </div>
}
```

### 3. Routes

You can also use feature flags to control the routing behavior in your .NET application. This allows you to enable or disable specific routes based on the state of a feature flag. To add feature flags to a route, you can use the `MapWhen` method provided by the feature management libraries. This method allows you to conditionally map a route based on the state of a feature flag. Here's an example of how to add a feature flag to a route:

```csharp
app.MapWhen(context => context.Request.Path.StartsWithSegments("/my-route") && Feature.IsEnabled("my-feature-flag"),
    builder =>
    {
        // Configure the route behavior
    });
```

### 4. Middleware

Feature flags can also be used to control the behavior of middleware components in your .NET application. This allows you to conditionally execute middleware based on the state of a feature flag. To add feature flags to middleware, you can use the `UseWhen` method provided by the feature management libraries. This method allows you to conditionally execute middleware based on the state of a feature flag.

Here's an example of how to add a feature flag to middleware:

```csharp
app.UseWhen(context => Feature.IsEnabled("my-feature-flag"),
    builder =>
    {
        // Configure the middleware behavior
    });
```

### 5. Use IFeatureManager

You can also use the `IFeatureManager` interface provided by the feature management libraries to check the state of a feature flag in your .NET application. This allows you to conditionally execute code based on the state of a feature flag. Here's an example of how to use the `IFeatureManager` interface:

```csharp
if (await featureManager.IsEnabledAsync("my-feature-flag"))
{
// Execute the code
}
```

By adding feature flags to different parts of your .NET application, you can easily control the availability of features without modifying the codebase. This promotes flexibility and enables teams to release features gradually and safely.

## Integrating with Azure App Configuration for feature flag management

Azure App Configuration provides a centralized and scalable solution for storing and managing configuration data, including feature flag settings. To integrate with Azure App Configuration for feature flag management, you need to configure your .NET application to use Azure App Configuration as a configuration source. This involves adding the necessary NuGet packages.

Here's a step-by-step guide on how to integrate .NET with App Configuration for feature flag management:

### 1. Add the required NuGet packages

To use App Configuration in your .NET application, you need to add the `Microsoft.Extensions.Configuration.AzureAppConfiguration` NuGet package to your project. This package provides the necessary dependencies for integrating .NET with App Configuration.

You can add the package using the .NET CLI:

```bash
.NET add package Microsoft.Extensions.Configuration.AzureAppConfiguration
```

### 2. Update the `appsettings.json`s File

Next, you need to update your `appsettings.json` file to include the connection string for your App Configuration instance. This connection string allows your .NET application to connect to the App Configuration service and retrieve feature flag settings.

### 3. Configure Azure App Configuration in `Startup.cs`

Finally, you need to configure App Configuration in your `Startup.cs` file. This involves adding the necessary code to retrieve feature flag settings from App Configuration and make them available in your .NET application.

Here's an example of how to configure App Configuration in your `Startup.cs` file:

```cs
public class Startup
{
    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAzureAppConfiguration(options =>
            options.Connect("<connection-string>")
                    .UseFeatureFlags(featureFlagOptions =>
                    {
                        featureFlagOptions.CacheExpirationInterval = TimeSpan.FromMinutes(5);
                    }));

        services.AddFeatureManagement();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseAzureAppConfiguration();
    }
}
```

With these configurations in place, your .NET application will retrieve feature flag settings from App Configuration, allowing you to manage feature flags centrally and scale your application's feature flag management.

## Conclusion

Feature flags are a powerful tool that allows developers to control feature availability and release features gradually and safely. By implementing feature flags in applications, you can promote continuous delivery, mitigate risks, and gather valuable user feedback. In this post, we have covered the basics of feature flags, explored how to add feature flags to different parts of your .NET application, and integrated with Azure App Configuration for centralized feature flag managements.

Now that you have a solid understanding of feature flags in .NET, you can leverage this powerful tool to enhance your software development process and deliver high-quality features to your users. Remember, feature flags are not just a toggle; they are a key enabler for modern software development practices. Embrace feature flags in your .NET applications and unlock the full potential of continuous delivery and user-centric development. Thanks so much for taking your time to read. Happy coding!
