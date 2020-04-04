---
title: "Building an identity server that supports OAuth 2.0 and OpenID Connect with ASP.NET Core and IdentityServer4 - Part 5"
date: 2020-04-04
tags: [C#, ASP.NET Core]
---

In the [previous post]({{ site.baseurl}}/identity-server-part-4) we added an MVC client to our project and protected it using our identity server. In this post we are going to add user registration functionality to our identity server before we move everything from in-memory database to a SQL Server database in the next post. I have to admit, there is not much going on in this post except for adding a few lines of code to our project. Let's open the `Server` project and get right into the code.

### RegisterViewModel

Add `Models/RegisterViewModel.cs` and add the following code therein:

```csharp
// Models/RegisterViewModel

using System.ComponentModel.DataAnnotations;

namespace Server.Models
{
    public class RegisterViewModel
    {
        [Required]
        public string Username { get; set; }

        [Required]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        [Required]
        [DataType(DataType.Password)]
        [Compare("Password")]
        public string ConfirmPassword { get; set; }

        public string ReturnUrl { get; set; }
    }
}
```

## AccountController

Let us go to `Controllers/AccountController.cs` and update it to add user registration endpoints:

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
        private readonly UserManager<IdentityUser> _userManager; // New

        public AccountController(
            SignInManager<IdentityUser> signInManager,
            UserManager<IdentityUser> userManager // New
            )
        {
            _signInManager = signInManager;
            _userManager = userManager; // New
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

        // New
        [HttpGet]
        public IActionResult Register(string returnUrl = null)
        {
            return View(new RegisterViewModel { ReturnUrl = returnUrl });
        }

        // New
        [HttpPost]
        public async Task<IActionResult> Register(RegisterViewModel viewModel)
        {
            if (!ModelState.IsValid)
            {
                return View(viewModel);
            }

            var user = new IdentityUser(viewModel.Username);
            var result = await _userManager.CreateAsync(user, viewModel.Password);

            if (result.Succeeded)
            {
                await _signInManager.SignInAsync(user, false);
                var returnUrl = viewModel.ReturnUrl ?? Url.Content("~/");
                return Redirect(returnUrl);
            }
            return View(viewModel);
        }
    }
}
```

## Register View

Add `Views/Account/Register.cshtml` and add the following code:

```html
@model RegisterViewModel

<form asp-controller="Account" asp-action="Register" method="post">
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
    <label>Confirm Password</label>
    <input asp-for="ConfirmPassword" />
    <span asp-validation-for="ConfirmPassword"></span>
  </div>
  <div>
    <button type="submit">Sign Up</button>
  </div>
</form>
<div>
  Already have an account?
  <a
    asp-controller="Account"
    asp-action="Login"
    asp-route-returnUrl="@Model.ReturnUrl"
    >Login</a
  >
</div>
```

## Update Login.cshtml

Open `Views/Account/Login.cshtml` and add a link to the registration page:

```html
@model LoginViewModel

<form asp-controller="Account" asp-action="Login" method="post">
  <input type="hidden" asp-for="ReturnUrl" />
  <!--So we can pass on the returnUrl-->
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
<!--New-->
<div>
  You don't have an account?
  <a
    asp-controller="Account"
    asp-action="Register"
    asp-route-returnUrl="@Model.ReturnUrl"
    >Register</a
  >
</div>
```

We are done. You can run the `Server` project and check if you can register. You may check out the source code on [GitHub](https://github.com/vince-nyanga/IdentityServerTutorial).

## Conclusion

In this very short post we added user registration to our indentity server. In the next post we are going to move all our data from an in-memory database to a SQL Server database. Thanks once again for taking your time to read.
