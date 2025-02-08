---
layout: post
title: "How to deploy an Angular 18+ as part of a .NET 8+ web app"
description: "All the details behind the actual integration between Angular and .NET when you try to deploy your app on a production environment."
category: Web development
tags: "Angular, .NET, ASP.NET"
---

ASP.NET Core ships with 
[Kestrel server](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/?view=aspnetcore-9.0&tabs=windows#kestrel-vs-httpsys), 
which is the default, cross-platform HTTP server.
Deploying a .NET 8+ with Angular 18+ is possible on different scenarios and we can also think of using a sort of encapsulation for
rendering the front-office app, containing the bare minimum to serve a Web-based app with HTML, JS and the minimal backend features, to 
render the app on a web browser. We can use Kestrel with a 
[reverse proxy server](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/when-to-use-a-reverse-proxy?view=aspnetcore-9.0) 
such as Nginx. 

![reverse-proxy-diagram](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/_static/kestrel-to-internet.png?view=aspnetcore-9.0)

<!--more-->

You can then attach much more pieces to your front-office app through synchronous or asynchronous communication techniques, for example
using gRPC, RESTful API and message passing. You will then have the ability to connect a set of microservices to your Web backend 
and display all the information through web pages generated using our beautiful Typescript Web framework, which is Angular. I do prefer
Angular to everything else because it's stable and well designed. Many frontend developers over the last decade have changed constantly
their development approach using tons of TS/JS libraries, and frankly it's like a Jungle out there. The value of a framework that will 
give you everything you need and that's basically self-contained is underestimated in the JS ecosystem.

You can achieve a better long-term result just focusing on your visual design, probably using Figma or a similar tool for your mockups,
and then implementing your Frontend in very reasonable and resilient manner using Angular. Then, to serve your HTTP 1/2/3 requests you
can rely upon a stable and well documented framework like:

* ASP.NET (using C# 8+)
* Sprint Boot (using Java or even [Scala](https://github.com/jecklgamis/spring-boot-scala-example))

In this blog post, we'll focus on the deployment of your C# 8 ASP.NET and Angular 18 app, to make the two frameworks work nicely
together.

## First step: packaging our Angular app into `wwwroot`
We need to package our frontend files taking advantage of `ng dist` in a way that can be actually served by ASP.NET Kestrel server.

We need to store everything produced by `ng build` into the `/wwwroot` directory inside our Backend project. Just to be absolutely clear let's say that we'll have something like that:

* `/backend/`: the folder that contains our ASP.NET backend files (e.g. `dotnet new webapi` generated files)
* `/frontend/`: the folder that contains all the Angular files (e.g. `ng new` generated files)

Now we want to put all the dist files generated during the building phase of our Angular app into `/backend/wwwroot`, which is the 
folder that by default will be served using our in-app web server.

Let's edit `angular.json` file in order to instruct Angular about that. Normally, Angular saves our dist files into `dist/frontend` and
then wraps all the generated files into a subfolder called `/browser`, but we don't want either.

```json
 "options": {
    "outputPath": {
       "base": "../API/wwwroot",
       "browser": ""
     },
```

We're changing architect/build node here. After Angular 17 we need to specify the base and the browser folder if we want to skip the
creation of the browser folder; note that now it's empty.

### Build the Angular app 
In order to build the Angular app we can now run the command `build` provided by Angular itself, but we can also specify the configuration we want to use to do so. In `angular.json` there is a section for those configurations and we can also find there the
so called allowed "budgets" in terms of the maximum kylobites that our app can actually reach for specific minified files like 
`main.js`. If you have any issue running the build command about the used budget, please double check this section of conf.

Let's build our app for development using the command `ng build -c development`. We can also run the same command for production and
the generated files will be minified, uglified and so on, but normally you want to build those only when the app is going to be 
build in a CI/CD pipeline ready to land on the staging/prod server, not before.

Now you should be able to see at the very least an `index.html` and a `main.js` files on your `backend/wwwroot` folder.

## Update `backend/Program.cs`
Now we need to add a middleware into our `Program.cs` in order to tell C# that we want to serve static files within our `wwwroot` folder. Please be extra careful because the ourdering of our middlewares here is paramount. **We must put `UseDefaultFiles()` and `UseStaticFiles()` only after our authentication and authorization**.

You should have something like that in your `Program.cs`:

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseDefaultFiles();  // Enables default file mapping on the current path
app.UseStaticFiles();   // Enables static file serving for the current request path

app.MapControllers();
```

Note that files are served from the path specified in `IWebHostEnvironment.WebRootPath` or `IWebHostEnvironment.WebRootFileProvider` which defaults to the `'wwwroot'` subfolder. Keep in mind that Angular is going to create distribution files for us that are actually static files.

## Test your app served directly by Kestrel and packed into ASP.NET
If you now try to run the app using `dotnet run` from your backend folder you should be able to see the Angular app browsing the local
URL of your backend app. That's nice, isn't it? :)

What is more the Angular app should be able to make you navigate by clicking on anchors and so on, BUT if you try to refresh the page on 
an angular route **the app will break**.

## Adding a FallbackController 
In order to have our single page application (SPA) being able to keep the Angular routing-based navigation active, we need to 
tell .NET to be always redirected to `/index.html` if the route is not defined. Essentially, we need a _fallback strategy_.

This new controller will be derived from the basic `Microsoft.AspNetCore.Mvc.Controller` rather than `AppController`. We need just the 
very basic functionality of an MVC Controller.

```csharp
public class FallbackController : Controller
{
    public ActionResult Index()
    {
        return PhysicalFile(
            Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "index.html"),
            "text/HTML"
        );
    }
}
```

Then in our `Program.cs`, after MapControllers we need to add:

 ```csharp
app.MapControllers();
app.MapFallbackToController("Index", "Fallback");
```
