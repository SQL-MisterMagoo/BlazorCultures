## Globalisation and Localisation in Blazor Server

Current documentation :
[ASP.NET Core Docs](https://docs.microsoft.com/en-us/aspnet/core/blazor/globalization-localization?view=aspnetcore-3.1#blazor-server)

This repo shows an exmaple of a working Glob/Loc Blazor project.

Try it out: https://blazorcultures.azurewebsites.net/
( Azure free tier, so don't expect speed! )

### Things to look at:

#### Enabling Localisation in startup.cs
##### Line 32 - add localization and tell it where to find the resx files
   
``` csharp
services.AddLocalization(options => options.ResourcesPath = "Resources");
```

##### Lines 48-51 - add supported cultures

``` csharp
var supportedCultures = new[] { "en-GB", "en-US", "fr" };
var localizationOptions = new RequestLocalizationOptions().SetDefaultCulture(supportedCultures[0])
	.AddSupportedCultures(supportedCultures)
	.AddSupportedUICultures(supportedCultures);
```

#### Resource files (resx)

We told the system to look in `Resources` so that's where to create the files.

You don't need a resx for your default culture - in this case `en-GB`, so there are just files for `fr` and `en-US`.

```
Pages.Index.en-US.resx
Pages.Index.fr.resx
```

*Note: The resource file names match the namespace of the class they are for - in this case the resources are for the Index page, whose namespace is Pages by default*

#### Host page

Because Blazor runs over SignalR it has no HTTP Requests to work with, so we add code to the Host page to set a cookie with the client culture.
SignalR will use the client side cookie to set the correct Culture.

##### Lines 3-4 - bring in the required namespaces

``` csharp
@using System.Globalization
@using Microsoft.AspNetCore.Localization
```

##### Lines 22-29 - create the cookie for SignalR

``` csharp
    @{
        this.HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }
```

#### Using localised strings in Index.razor

##### Line 2 - Inject a String Localizer from DI for the Index

``` csharp
@inject Microsoft.Extensions.Localization.IStringLocalizer<Index> text
```

##### Line 5 - use a localized string

```
<h1>@text["Hello, world!"]</h1>
```

*Note that the string "Hello, world!" is both the default value and the lookup key in the resource files.*

#### Bonus Feature - Main Layout - Choose a Culture

I've added some code to allow the user to manually choose a supported culture.
If they change culture, we set the cookie and reload the page.

#### Bonus Feature - Query String Culture

By default ASP.NET Core configures a Querystring culture provider, so you can set the culture in the browser address bar.

Try adding `/?culture=fr` to the URL to set French culture.

#### Bonus Feature - Timezone Offset in App.razor

Timezones are hard. I like to get the UTC offset from the client and share it as a CascadingValue.

##### Lines 4-6 - Cascading Value

I have wrapped the RouteView in a cascading value that share the time offset to any component that wants it.

##### Lines 14-26 Get the offset from the client

These lines simply use JS Interop to get the client browser UTC offset.
It's crude, but works a lot of the time as users tend to have their system clocks set appropriately for them, regardless of what timezone might be reported by the system.



 
##### 

