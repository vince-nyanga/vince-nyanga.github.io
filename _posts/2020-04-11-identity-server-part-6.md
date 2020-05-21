---
title: "Building an identity server that supports OAuth 2.0 and OpenID Connect with ASP.NET Core and IdentityServer4 - Part 6"
date: 2020-04-11
categories: [C#, ASP.NET Core]
---

In the [previous post]({{ site.baseurl}}/identity-server-part-4) we added user registration to our identity server project. Now that we have most of the functionality working we can now move everything from an in-memory database to a SQL Server database. I relied heavily on the [documentation](http://docs.identityserver.io/en/release/quickstarts/8_entity_framework.html) in this post so I encourage you to check it out for more information. Let's get started.

## Install Packages

Add the following NuGet packages to the `Server` project:

```
IdentityServer4.EntityFramework
```

```
Microsoft.EntityFrameworkCore.Design
```

```
Microsoft.EntityFrameworkCore.SqlServer
```

In addition, open the `Server.csproj` file and add the following tool reference that provides EF Core command line tools that we are going to use:

```xml
<ItemGroup>
    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="[LATEST-VERSION]" />
</ItemGroup>
```

## The Code

Now let's go to the `appsettings.json` file and add a database connection:

```json
{
"ConnectionStrings": {
    "DefaultConnection": "Data Source=(LocalDb)\\MSSQLLocalDB;Initial Catalog=IdentityServerDB;Trusted_Connection=yes;"
  },
...
}
```

I am making an assumption that you have a local SQL Server database installed on your machine. Let us now update the `Startup.cs` class so that our configuration can use a SQL Server database.

```csharp
// Startup.cs
public class Startup
    {
        // New
        private readonly IConfiguration _configuration;

        // New
        public Startup(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        public void ConfigureServices(IServiceCollection services)
        {
            // New
            var connectionString = _configuration.GetConnectionString("DefaultConnection");

            services.AddDbContext<AppDbContext>(config =>
            {
                config.UseSqlServer(connectionString);
            });

            services.AddIdentity<IdentityUser, IdentityRole>(config =>
            {
                config.Password.RequiredLength = 5;
                config.Password.RequireDigit = false;
                config.Password.RequireUppercase = false;
                config.Password.RequireNonAlphanumeric = false;
            })
                .AddEntityFrameworkStores<AppDbContext>()
                .AddDefaultTokenProviders();

            services.ConfigureApplicationCookie(config =>
            {
                config.Cookie.Name = "Server.Cookie";
                config.LoginPath = "/Account/Login";
            });

            var migrationsAssembly = typeof(Startup).Assembly.GetName().Name; // New

            services.AddIdentityServer()
                .AddAspNetIdentity<IdentityUser>()
                .AddConfigurationStore(options => // New
                {
                    options.ConfigureDbContext = builder =>
                        builder.UseSqlServer(connectionString,
                            sql => sql.MigrationsAssembly(migrationsAssembly));
                })
                .AddOperationalStore(options => // New
                {
                    options.ConfigureDbContext = builder =>
                        builder.UseSqlServer(connectionString,
                            sql => sql.MigrationsAssembly(migrationsAssembly));
                })
                 .AddDeveloperSigningCredential();

             services.AddControllersWithViews();

        }
        // ....

    }

```

Now that our identity server is configured to save configuration and operation data in a SQL Server database, let's add database migrations. Open the terminal or command line and run the following commands:

1. Add migration for `PersistedGrantDbContext`:

```
dotnet ef migrations add InitialIdentityServerPersistedGrantDbMigration -c PersistedGrantDbContext -o Data/Migrations/IdentityServer/PersistedGrantDb
```

2. Add migration for identity server `ConfigurationDbContext`:

```
dotnet ef migrations add InitialIdentityServerConfigurationDbMigration -c ConfigurationDbContext -o Data/Migrations/IdentityServer/ConfigurationDb
```

3. Add migration for our own `AppDbContext`:

```
dotnet ef migrations add InitialAppDbContextDbMigration -c AppDbContext -o Data/Migrations/AppContext
```

If all migrations were added without a problem let's now update the database using the following commands:

1. `PersistedGrantDbContext`:

```
dotnet ef database update -c PersistedGrantDbContext
```

2. `ConfigurationDbContext`:

```
dotnet ef database update -c ConfigurationDbContext
```

3. `AppDbContext`:

```
dotnet ef database update -c AppDbContext
```

You can check your database object explorer to confirm if the database and all the relevant tables were created.

Finally, let's go to `Program.cs` file and add code that will seed our database with the configuration data that was stored inside our `Configuration.cs` class:

```csharp
// Program.cs

using IdentityServer4.EntityFramework.DbContexts;
using IdentityServer4.EntityFramework.Mappers;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System.Linq;

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
                try
                {
                    var userManager = scope.ServiceProvider
                        .GetRequiredService<UserManager<IdentityUser>>();
                    var user = new IdentityUser("vince");
                    userManager.CreateAsync(user, "pass123").GetAwaiter().GetResult();

                }catch (Exception e)
                {
                    // ignore
                }

                // New

                // Seeding for IdentityServer
                scope.ServiceProvider.GetRequiredService<PersistedGrantDbContext>().Database.Migrate();

                var context = scope.ServiceProvider.GetRequiredService<ConfigurationDbContext>();

                context.Database.Migrate();

                if (!context.Clients.Any())
                {
                    foreach (var client in Configuration.Clients)
                    {
                        context.Clients.Add(client.ToEntity());
                    }
                    context.SaveChanges();
                }

                if (!context.IdentityResources.Any())
                {
                    foreach (var resource in Configuration.IdentityResources)
                    {
                        context.IdentityResources.Add(resource.ToEntity());
                    }
                    context.SaveChanges();
                }

                if (!context.ApiResources.Any())
                {
                    foreach (var resource in Configuration.Apis)
                    {
                        context.ApiResources.Add(resource.ToEntity());
                    }
                    context.SaveChanges();
                }
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

We are done! You may run the `Server` project and see your SQL Server database tables being populated with the identity server configuration. The advantage that we get from persisting configuration to a database is that we won't need to restart the application everytime we make a configuration change. You may view the complete project on [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial).

## Conclusion

In this post we moved our identity server configuration as well as our user Identity to a SQL Server database. This is going to be the last post in this series for now. Like always, thanks so much for taking your time to read. Stay safe and God bless.
