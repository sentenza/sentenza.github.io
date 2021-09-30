---
layout: post
title: "Build a modern RESTful application with Scala Play and Angular 7+"
description: "How to build, from scratch a web app using Angular and the Scala Play framework"
category: scala
tags: [Play, Angular]
---

In this post we'll go through all the essential steps that will eventually lead you to the construction of a new self-contained web application based on a RESTful API built with 
the [Scala Play framework](https://www.playframework.com/documentation/2.7.x/Introduction) and consumed by an [Angular 7](https://angular.io) front end. 
We will build the API from scratch putting the front end code in the same repository (within the `/ui` folder).

A complete example can be also found at [gitlab.com/alfredotorre/ngscalaplay](https://gitlab.com/alfredotorre/ngscalaplay)

---


The RESTful API will:

- be secured using access tokens (no data without a valid access token)
- return a list of users
- return a list of posts

The Front-End will:

- contain all the views and it will serve them as a <abbr title="Single Page Application">SPA</abbr>
- enable a complete de-coupling between the view layer and the API

Targeted versions:

- Scala Play framework **v2.7.x**
- Angular **v7.2+**
- [sbt][sbt] **v1.2.8**
- Scala v2.12.8

## Project setup

We're going to init a new Scala Play project using [sbt][sbt] and a [Giter8](http://www.foundweekends.org/giter8/) template provided by [Lightbend](https://www.lightbend.com) itself. Run the following command to scaffold the new basic Scala Play project:

```
sbt new playframework/play-scala-seed.g8
```

*g8* will prompt you for a valid organization name, so feel free to use your own domain or something like `test.scalaplay`. In my case it will be `it.ingegnati`. Take your time to look into this new Play project, you will eventually find a controller and a `scala.html` view; well, we're going to clean everything up to swap the Angular framework in. 

```bash
# Add .gitkeep to keep tracking empty folders
touch app/controllers/.gitkeep
touch app/views/.gitkeep
touch public/.gitkeep
touch test/controllers/.gitkeep

# Remove unused folders
rm -rf app/controllers/HomeController.scala app/views/*.html public/* test/controllers/HomeControllerSpec.scala

# Clear out the routes, as they'll no longer compile
echo "" > conf/routes
```

Now we can init a new Angular project within the `./ui` directory using the Angular CLI command [`ng new`](https://angular.io/cli/new):

```bash
ng new ng-scala-play --directory=ui --style=scss
```

Also, you might want to update your brand new ng app to the latest version (at the moment I'm on v2.7.2) of the framework:

```bash
cd ui/
ng update @angular/core
ng update @angular/cli
npm install --save-dev karma@4.0.1
npm install
```

If everything looks good we can now configure our Scala application to build and serve both the frontend and the backend using Play static routes to serve the FE and make it communicate with the API through a RESTful interface. 

### Combine Frontend and Backend development servers

In order to do that I'm going to follow the same approach described on [this blog post](https://medium.com/@yohan.gz/https-medium-com-yohan-gz-angular-with-play-framework-a6c3f8b339f3), hence we need to edit the _scripts_ defined in `package.json` in order to target `public/` when building the Angular application, becase we want the Play framework to take care of serving these files using the `FrontendController`, that we'll create in a moment. 

```json
{
  ...
  "scripts": {
    ...
    "start": "ng serve --open --proxy-config src/proxy.conf.js --host 0.0.0.0",
    "test": "ng test --watch",
    "test:ci": "ng test --watch=false --browsers ChromeHeadless",
    "test:coverage": "ng test --watch=true --code-coverage=true",
    "test:coverage:ci": "ng test --watch=false --code-coverage=true --browsers ChromeHeadless",
    "build:dev": "ng build --progress --output-path ../public",
    "build:prod": "ng build --progress --prod --output-path ../public"
  }
}
```

Another important piece to pay attention to is the "start" script which will use `ui/src/proxy.conf.js`:

```javascript
const PROXY_CONFIG = {
    "**": {
      "target": "http://localhost:9000",
      "secure": false,
      "bypass": function (req) {
        if (req && req.headers && req.headers.accept && req.headers.accept.indexOf("html") !== -1) {
          console.log("Skipping proxy for browser request.");
          return "/index.html";
        }
      }
    }
  };
  
  module.exports = PROXY_CONFIG;
```

Basically, we're creating a reverse proxy using the frontend development server to forward certain paths to another server (`http://localhost:9000`) and make the results available. Keep in mind that your Angular dev server will serve `http://localhost:4200` by default. See also [this blog post](https://vsupalov.com/combine-frontend-and-backend-development-servers/) for more information. 

### Using sbt to run both the backend and the frontend

Now we need to wire up our sbt project with Angular and the npm scripts we defined above. To do so we have to create `/project/FrontendCommands.scala` in order to have access to all the frontend build associated commands from sbt:

```scala
/**
  * Frontend build commands.
  * Change these if you are using some other package manager. i.e: Yarn
  */
object FrontendCommands {
  val dependencyInstall: String = "npm install"
  val test: String = "npm run test:ci"
  val serve: String = "npm run start"
  val build: String = "npm run build:prod"
}
```

We can now hook all these commands when Play runs in dev mode defining an implementation of [PlayRunHook](https://www.playframework.com/documentation/2.7.x/SBTCookbook#Hooking-into-Plays-dev-mode), to do so include `/project/FrontendRunHook.scala`:

```scala
import java.net.InetSocketAddress

import play.sbt.PlayRunHook
import sbt._

import scala.sys.process.Process

/**
  * Frontend build play run hook.
  * @link https://www.playframework.com/documentation/2.7.x/SBTCookbook
  * When Play runs in dev mode (sbt run) we can start up additional processes 
  * that are required for development.
  * This can be done by defining a PlayRunHook, which is a trait with the following methods:
  * - beforeStarted(): Unit - called before the play application is started, but after all “before run” 
  *      tasks have been completed.
  * - afterStarted(): Unit - called after the play application has been started.
  * - afterStopped(): Unit - called after the play process has been stopped.
  */
object FrontendRunHook {
  def apply(base: File): PlayRunHook = {
    object UIBuildHook extends PlayRunHook {

      var process: Option[Process] = None

      /**
        * Change these commands if you want to use Yarn.
        */
      var npmInstall: String = FrontendCommands.dependencyInstall
      var npmRun: String = FrontendCommands.serve

      // Windows requires npm commands prefixed with cmd /c
      if (System.getProperty("os.name").toLowerCase().contains("win")){
        npmInstall = "cmd /c" + npmInstall
        npmRun = "cmd /c" + npmRun
      }

      /**
        * Executed before play run start.
        * Run npm install if node modules are not installed.
        */
      override def beforeStarted(): Unit = {
        if (!(base / "ui" / "node_modules").exists()) Process(npmInstall, base / "ui").!
      }

      /**
        * Executed after play run start.
        * Run npm start
        */
      override def afterStarted(): Unit = {
        process = Option(
          Process(npmRun, base / "ui").run
        )
      }

      /**
        * Executed after play run stop.
        * Cleanup frontend execution processes.
        */
      override def afterStopped(): Unit = {
        process.foreach(_.destroy())
        process = None
      }

    }

    UIBuildHook
  }
}
```

And also add `/ui-build.sbt`:

```scala
import scala.sys.process.Process

/*
 * UI Build hook Scripts
 */

// Execution status success.
val Success = 0

// Execution status failure.
val Error = 1

// Run angular serve task when Play runs in dev mode, that is, when using 'sbt run'
// https://www.playframework.com/documentation/2.7.0/SBTCookbook
PlayKeys.playRunHooks += baseDirectory.map(FrontendRunHook.apply).value

// True if build running operating system is windows.
val isWindows = System.getProperty("os.name").toLowerCase().contains("win")

// Execute on commandline, depending on the operating system. Used to execute npm commands.
def runOnCommandline(script: String)(implicit dir: File): Int = {
  if(isWindows){ Process("cmd /c " + script, dir) } else { Process(script, dir) } }!

// Check of node_modules directory exist in given directory.
def isNodeModulesInstalled(implicit dir: File): Boolean = (dir / "node_modules").exists()

// Execute `npm install` command to install all node module dependencies. Return Success if already installed.
def runNpmInstall(implicit dir: File): Int =
  if (isNodeModulesInstalled) Success else runOnCommandline(FrontendCommands.dependencyInstall)

// Execute task if node modules are installed, else return Error status.
def ifNodeModulesInstalled(task: => Int)(implicit dir: File): Int =
  if (runNpmInstall == Success) task
  else Error

// Execute frontend test task. Update to change the frontend test task.
def executeUiTests(implicit dir: File): Int = ifNodeModulesInstalled(runOnCommandline(FrontendCommands.test))

// Execute frontend prod build task. Update to change the frontend prod build task.
def executeProdBuild(implicit dir: File): Int = ifNodeModulesInstalled(runOnCommandline(FrontendCommands.build))


// Create frontend build tasks for prod, dev and test execution.

lazy val `ui-test` = TaskKey[Unit]("run ui tests when testing application.")

`ui-test` := {
  implicit val userInterfaceRoot = baseDirectory.value / "ui"
  if (executeUiTests != Success) throw new Exception("ui tests failed!")
}

lazy val `ui-prod-build` = TaskKey[Unit]("run ui build when packaging the application.")

`ui-prod-build` := {
  implicit val userInterfaceRoot = baseDirectory.value / "ui"
  if (executeProdBuild != Success) throw new Exception("oops! ui build crashed.")
}

// Execute frontend prod build task prior to play dist execution.
dist := (dist dependsOn `ui-prod-build`).value

// Execute frontend prod build task prior to play stage execution.
stage := (stage dependsOn `ui-prod-build`).value

// Execute frontend test task prior to play test execution.
test := ((test in Test) dependsOn `ui-test`).value
```

## Adding the Frontend and the API controllers

To start, create a new file in the `app/controllers` folder called `FrontendController.scala`:

```bash
touch app/controllers/FrontendController.scala
```

Next, populate `FrontendController.scala` with the following:

```scala
// app/controllers/FrontendController.scala

package controllers

import javax.inject._

import play.api.Configuration
import play.api.http.HttpErrorHandler
import play.api.mvc._

/**
  * Frontend controller managing all static resource associate routes.
  * @param assets Assets controller reference.
  * @param errorHandler HttpErrorHandler
  * @param config Configuration
  * @param cc Controller components reference.
  */
@Singleton
class FrontendController @Inject()(
    assets: Assets,
    errorHandler: HttpErrorHandler,
    config: Configuration,
    cc: ControllerComponents
    ) extends AbstractController(cc) {

  def index: Action[AnyContent] = assets.at("index.html")

  def assetOrDefault(resource: String): Action[AnyContent] = if (resource.startsWith(config.get[String]("apiPrefix"))){
    Action.async(r => errorHandler.onClientError(r, NOT_FOUND, "Not found"))
  } else {
    if (resource.contains(".")) assets.at(resource) else index
  }
}
```

Edit `conf/routes` as follows:


```
# Serve index page from public directory
GET     /                           controllers.FrontendController.index()

# An example route (Prefix all API routes with apiPrefix defined in application.conf)
GET     /api/summary                controllers.HomeController.appSummary

# Serve static assets under public directory
GET /*file controllers.FrontendController.assetOrDefault(file)

# Map static resources from the /public folder to the /assets URL path
# GET     /assets/*file               controllers.Assets.versioned(path="/public", file: Asset)
```

## Test the Frontend scripts

The available commands are:

```
    sbt clean           # Clean existing build artifacts

    sbt stage           # Build your application from your project’s source directory

    sbt run             # Run both backend and frontend builds in watch mode

    sbt dist            # Build both backend and frontend sources into a single distribution artifact

    sbt test            # Run both backend and frontend unit tests

```

At first I would suggest to clean all the sbt artifacts, and then to try building the FE of your application for production. To ensure that the FE is working as expected move into `/ui` and then execute:

```
npm install
npm run build:dev
npm run build:prod
```

If everything works it's possible to run both the backend and the frontend server on the background using just `sbt run`.

Essentially, when you give **`sbt run`**:

1. sbt will eventually look into `ui-build.sbt`
2. PlayRunHooks will also run `FrontendRunHook.apply`
3. `FrontendCommands.serve` will be called
4. `npm run start` will be executed

## Add an API Controller

Create a new file in the `app/controllers` folder called `ApiController.scala`:

```
touch app/controllers/ApiController.scala
```

Next, populate `ApiController.scala` with the following:

```scala
// app/controllers/ApiController.scala

// Make sure it's in the 'controllers' package
package controllers

import javax.inject.{Inject, Singleton}
import play.api.mvc.{AbstractController, ControllerComponents}

@Singleton
class ApiController @Inject()(cc: ControllerComponents)
  extends AbstractController(cc) {

  // Create a simple 'ping' endpoint for now, so that we
  // can get up and running with a basic implementation
  def ping = Action { implicit request =>
    Ok("PONG!")
  }

}
```

To give it a test run, open another terminal window and use the curl command to issue a request to our ping endpoint:

`curl localhost:9000/api/ping`


The full project can be found at [gitlab.com/alfredotorre/ngscalaplay](https://gitlab.com/alfredotorre/ngscalaplay)


### Resources

- [Auth0: Build and Secure APIs with Scala and the Play Framework](https://auth0.com/blog/build-and-secure-a-scala-play-framework-api/)
- [Angular 7 with Play Framework 2.6.x](https://medium.com/@yohan.gz/https-medium-com-yohan-gz-angular-with-play-framework-a6c3f8b339f3)
- [conf/application.conf template](https://github.com/yohangz/scala-play-angular-seed/blob/master/conf/application.conf)
- [Play framework: enable the CORS filter](https://www.playframework.com/documentation/2.7.x/CorsFilter)
- [Pre-flight calls and OPTIONS - CORS](https://stackoverflow.com/a/29954326/1977778)
- [Alvin Alexander: How to write a Play Framework POST request web service](https://alvinalexander.com/scala/how-to-write-play-framework-http-post-request-json-web-service)
- [Play Framework Authentication in a single page app](https://stackoverflow.com/questions/39865650/play-framework-authentication-in-a-single-page-app)
- [Play! Framework JSON & Scala Example](https://examples.javacodegeeks.com/enterprise-java/play-framework-json-scala-example/)
- [Gist: BasicAuthFilter.scala](https://gist.github.com/EdgeCaseBerg/e4bdec2c2907f40b4b72)

[sbt]: https://www.scala-sbt.org/download.html

