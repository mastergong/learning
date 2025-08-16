# Architecture 01 — ASP.NET Core MVC พื้นฐาน (.NET 9)

## โครงสร้างโปรเจกต์ MVC
```
MyMvc/
  Controllers/
  Models/
  Views/
  Program.cs
```

## Program.cs (Minimal Hosting)
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## Controller + View
```csharp
// Controllers/HomeController.cs
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        ViewData["Message"] = "Hello MVC on .NET 9";
        return View();
    }
}
```

```html
@* Views/Home/Index.cshtml *@
<h1>@ViewData["Message"]</h1>
```

## Model Binding & Validation
```csharp
public record RegisterVm
{
    [Required, EmailAddress]
    public string Email { get; init; } = string.Empty;
    [Required, MinLength(6)]
    public string Password { get; init; } = string.Empty;
}

public class AccountController : Controller
{
    [HttpGet]
    public IActionResult Register() => View();

    [HttpPost]
    public IActionResult Register(RegisterVm vm)
    {
        if (!ModelState.IsValid) return View(vm);
        // TODO: Save
        TempData["ok"] = "Registered!";
        return RedirectToAction("Index", "Home");
    }
}
```

## Run
```
dotnet new mvc -n MyMvc
dotnet run --project MyMvc
```
