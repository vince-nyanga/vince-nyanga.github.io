---
title: "Building an identity server that supports OAuth 2.0 and OpenID Connect with ASP.NET Core and IdentityServer4 - Part 3"
date: 2020-03-23
tags: [C#, ASP.NET Core]
---

In the [previous post]({{ site.baseurl}}/identity-server-part-2) we implemented a protected REST API and a console application that consumed the API. The console application made use of an access token issued by our identity server to authorize its requests to the API. In this post we are going to add more functionality to our indentity server. We are going to add support for users to login with their usernames and passwords. Let's get started.

## Install Required Packages

Before we proceed let's install the following NuGet packages to our `Server` project:

- `IdentityServer4.AspNetIdentity`
- `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore`
- `Microsoft.EntityFrameworkCore.InMemory`

## The Code

Now let's write some code. Let us add a database context for our `IdentityUser`. I am assuming you have some knowledge of EntityFramework and ASP.NET Core Identity. Create a file `Data/AppDbContext.cs` and add the following code:

```csharp
// Data/AppDbContext.cs

using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace Server.Data
{
    public class AppDbContext : IdentityDbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options)
            :base(options)
        {

        }
    }
}

```

Now that we have a database context for our users, let us go to `Startup.cs` and wire up everything. Add the following code to your `Startup.cs` file:

```csharp
// Startup.cs

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Server.Data;

namespace Server
{
    public class Startup
    {

        public void ConfigureServices(IServiceCollection services)
        {
            // New
            services.AddDbContext<AppDbContext>(config =>
            {
                config.UseInMemoryDatabase("ServerDb");
            });

            // New
            services.AddIdentity<IdentityUser, IdentityRole>(config =>
            {
                config.Password.RequiredLength = 5;
                config.Password.RequireDigit = false;
                config.Password.RequireUppercase = false;
                config.Password.RequireNonAlphanumeric = false;
            })
                .AddEntityFrameworkStores<AppDbContext>()
                .AddDefaultTokenProviders();

            // New
            services.ConfigureApplicationCookie(config =>
            {
                config.Cookie.Name = "Server.Cookie";
                config.LoginPath = "/Account/Login";
            });

            services.AddIdentityServer()
                .AddAspNetIdentity<IdentityUser>() // New
                .AddInMemoryApiResources(Configuration.Apis)
                .AddInMemoryClients(Configuration.Clients)
                .AddDeveloperSigningCredential();

            services.AddControllersWithViews();

        }


        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseIdentityServer();


            app.UseEndpoints(endpoints =>
            {
                endpoints.MapDefaultControllerRoute();
            });
        }
    }
}

```

Let us now go to the `Program.cs` file and seed our user database with a user that we will use to test our login functionality.

```csharp
// Program.cs

using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace Server
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();

            // seed with user
            using(var scope = host.Services.CreateScope())
            {
                var userManager = scope.ServiceProvider
                    .GetRequiredService<UserManager<IdentityUser>>();
                var user = new IdentityUser("vince");
                userManager.CreateAsync(user, "pass123").GetAwaiter().GetResult();
            }
            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}

```

## User Interface

Now let us turn our attention to the UI. First, create `Models/LoginViewModel.cs` and add the following code:

```csharp
// Models/LoginViewModel.cs

using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Threading.Tasks;

namespace Server.Models
{
    public class LoginViewModel
    {
        [Required]
        public string Username { get; set; }

        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        public string ReturnUrl { get; set; }
    }
}

```

Let's now add our controller -- `Controllers/AccountController.cs` and add the following:

```csharp
// Controllers/AccountController.cs

using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Server.Models;
using System.Threading.Tasks;

namespace Server.Controllers
{
    public class AccountController : Controller
    {
        private readonly SignInManager<IdentityUser> _signInManager;

        public AccountController(SignInManager<IdentityUser> signInManager)
        {
            _signInManager = signInManager;
        }

        [HttpGet]
        public IActionResult Login(string returnUrl=null)
        {
            return View(new LoginViewModel { ReturnUrl = returnUrl});
        }

        [HttpPost]
        public async Task<IActionResult> Login(LoginViewModel viewModel)
        {
            if (!ModelState.IsValid)
            {
                return View(viewModel);
            }

            var result = await _signInManager.PasswordSignInAsync(viewModel.Username, viewModel.Password, false, false);

            if (result.Succeeded)
            {
                var returnUrl = viewModel.ReturnUrl ?? Url.Content("~/");
                return Redirect(returnUrl);
            }

            return View(viewModel);
        }
    }
}

```

Now that we are done with our controller let's add our login view. Add `Views/Account/Login.cshtml` and add the following:

```html
<!--Views/Account/Login.cshtml-->

@model LoginViewModel

<form asp-controller="Account" asp-action="Login" method="post">
  <!--So we can pass on the returnUrl-->
  <input type="hidden" asp-for="ReturnUrl" />
  <div>
    <label>Username</label>
    <input asp-for="Username" />
    <span asp-validation-for="Username"></span>
  </div>
  <div>
    <label>Password</label>
    <input asp-for="Password" />
    <span asp-validation-for="Password"></span>
  </div>
  <div>
    <button type="submit">Login</button>
  </div>
</form>
```

Note that we are not going to talk about the `_ViewImports.cshtml` file in this post. You can add it and copy the contents therein from [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial/blob/master/Server/Views/_ViewImports.cshtml). We are done. You may run your server project and navigate to `Account/Login`. After successfully loging in you will be redirected to the home page that does not exist at the moment. The complete source code can be found on [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial).

## Conclusion

In this post we added support for users to login into our identity server with their username and password. This is gonna come in handy in our next post where we are going to create an MVC client that will need users to login to access protected views. Till next time, take care and thanks for reading.
