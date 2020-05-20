---
title: "Building an identity server that supports OAuth 2.0 and OpenID Connect with ASP.NET Core and IdentityServer4 - Part 4"
date: 2020-03-28
categories: [C#, ASP.NET Core]
---

In the [previous post]({{ site.baseurl}}/identity-server-part-3) we added support for users to login to our identity server using ASP.NET Core `Identity` and `Entity Framework`. In this post we are going to add a simple MVC client that will make use of our identity server to protect its resources. Let's get started.

## What We Need To Achieve

Our MVC client is going to have some protected views that require users to be authenticated. Instead of implementing this functionality inside the MVC app, we make use of the identity server that we built. When a user navigates to a protected view they will be redirected to the identity server to login. Once they have successfully logged in they will then redirected back to the protected view.

## The Code

Add a new empty web app called `MvcClient` to the solution and add the following package that we are going to need:

```
Microsoft.AspNetCore.Authentication.OpenIdConnect
```

Now let us go to the `Startup.cs` class and add the following code:

```csharp
// Startup.cs

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace MvcClient
{
    public class Startup
    {


        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews();

            services.AddAuthentication(config =>
            {
                config.DefaultScheme = "Cookie";
                config.DefaultChallengeScheme = "oidc";
            })
                .AddCookie("Cookie")
                .AddOpenIdConnect("oidc", config =>
                {
                    config.Authority = "[IDENTITY_SERVER_URL_HERE]";
                    config.ClientId = "mvc_client";
                    config.ClientSecret = "mvc_client_secret";
                    config.SaveTokens = true; // persist tokens in the cookie
                    config.ResponseType = "code";
                });
        }


        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();
            app.UseAuthentication();
            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapDefaultControllerRoute();
            });
        }
    }
}

```

We added authentication services to DI inside the `ConfigureServices` method. We are making use of a cookie to locally sign in the user and set the `DefaultChallengeScheme` to `"oidc"` because we want to make use of the OpenID Connect protocol when the user logs in. The `AddOpenIdConnect` method is used to configure the handler that will perform the OpenID Connect protocol. Inside that method we configure the authority and identify our client using the client id.

Let us now add a controller for our MVC app. Create `Controllers/HomeController.cs` and add the following code to it:

```csharp
// Controllers/HomeController.cs

using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace MvcClient.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }

        [Authorize]
        public IActionResult Protected()
        {
            return View();
        }
    }
}

```

We have a protected endpoint (`Protected()`) that will require a user to be authenticated. Let's go ahead and add our Views:

```html
<!--Views/Home/Index.cshtml-->
<h1>Hello World!</h1>
```

```html
<!--Views/Home/Protected.cshtml-->
<h1>Top Secret!!!</h1>
<p>I love ASP.NET Core</p>
```

We are done with the MVC client. Let us now go to the `Server` project and make a few additions to the code. First, open the `Configuration.cs` class and add this:

```csharp
// Configuration.cs

public static class Configuration
{
     // New
    public static IEnumerable<IdentityResource> IdentityResources =>
        new List<IdentityResource>
        {
            new IdentityResources.OpenId(),
            new IdentityResources.Profile()
        };

    // ...

}

```

We have added a list of identity resources that our identity server is going to protect. These resources are going to be required by our MVC client as per the OpenID Connect protocol.

While we are still inside the `Configuration.cs` class let us register our MVC client to the identity server by adding it to `Clients` list:

```csharp
// Configuration.cs
public static IEnumerable<Client> Clients =>
    new List<Client>
    {
        new Client
        {
            ClientId = "console_app",
            ClientSecrets = { new Secret("console_app_secret".ToSha256())},
            AllowedGrantTypes = GrantTypes.ClientCredentials,
            AllowedScopes = { "dummy_api" }
        },
        // New
        new Client
        {
            ClientId = "mvc_client",
            ClientSecrets = { new Secret("mvc_client_secret".ToSha256()) },
            AllowedGrantTypes = GrantTypes.Code,
            RequireConsent = false,
            AllowedScopes =
            {
                "dummy_api",
                IdentityServerConstants.StandardScopes.OpenId,
                IdentityServerConstants.StandardScopes.Profile
            },
             RedirectUris = { "[MVC_CLIENT_URL_HERE]/signin-oidc" }
        }
    };

```

Now let's go to the `Startup.cs` class and register the identity resources that we defined earlier.

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // ... left out for brevity

    services.AddIdentityServer()
        .AddAspNetIdentity<IdentityUser>()
        .AddInMemoryApiResources(Configuration.Apis)
        .AddInMemoryClients(Configuration.Clients)
        .AddInMemoryIdentityResources(Configuration.IdentityResources) // New
        .AddDeveloperSigningCredential();

    // ...

}

```

We are done. Now run both the `Server` and the `MvcClient` projects and inside the MVC client navigate to `Home/Protected`. You will be redirected to the identity server's login page where you will log in using the user credentials we added in the previous post. After successfully loging in you will then be redirected back to the protected page to view the top secret message. Tokens from the identity server will be stored inside the cookie that we configured earlier. We have accomplished our mission for this post. You may check out the source code on [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial).

## Conclusion

In this post we added an MVC client to our project. The client has a protected view that requires a user to be authenticated first. Instead of implementing this functionality inside the MVC app itself, we outsource it to the identity server that we built. When the user navigates to the protected view they will be redirected to the identity server's login page that we created in the previous post. When they have successfully logged in they will be redirected back to view the protected page. Once again, thanks so much for reading and I hope and pray that you're safe from the **Covid-19** virus.
