# Request的生命周期

- [概述](#overview)
- [Request Lifecycle](#request-lifecycle)
- [启动文件](#start-files)
- [应用程序事件](#application-events)

<a name="overview"></a>
## 概述

在现实世界中使用工具时，如果理解了工具的工作原理，使用起来就会更加有底气。应用开发也是如此。当你理解了开发工具是如何工作的，使用起来就会更加自如。这篇文档的目标就是提供一个高层次的概述，使你对于Laravel框架的运行方式有一个较好的把握。在更好地了解了整个框架之后，框架的组件和功能就不再显得那么神秘，开发起应用来也更加得心应手。这篇文档包含了关于请求生命周期的高层次概述，以及启动文件和应用程序事件的相关内容。

如果你不能立即理解所有的术语，别灰心，可以先有一个大致的把握，在阅读文档其他章节的过程中继续积累和消化知识。

<a name="request-lifecycle"></a>
## 请求生命周期

发送给应用程序的所有请求都经由 `public/index.php` 脚本处理。如果使用的是 Apache 服务器，Laravel中包含的 `.htaccess` 文件将对所有请求进行处理并传递给 `index.php`。这是Laravel从接受客户端请求到返回响应给客户端的整个过程的开始。若能对于Laravel的引导过程(bootstrap process)有一个大致的认识，将有助于理解框架，我们不妨先讨论这个。

到目前为止，学习Laravel引导过程所需掌握的最重要的概念就是 **服务提供器**。打开 `app/config/app.php` 配置文件，找到 `providers` 数组，你会发现一个服务提供器的列表。这些提供器充当了Laravel的主要引导机制。在我们深入服务提供器之前，先回到 `index.php`的讨论。当一个请求进入 `index.php` 文件，`bootstrap/start.php` 文件会被加载。这个文件会创建一个 Laravel `Application` 对象，该对象同时作为框架的 [IoC 容器](/docs/ioc)。

After creating the `Application` object, a few project paths will be set and [environment detection](/docs/configuration#environment-configuration) will be performed. Then, an internal Laravel bootstrap script will be called. This file lives deep within the Laravel source, and sets a few more settings based on your configuration files, such as timezone, error reporting, etc. But, in addition to setting these rather trivial configuration options, it also does something very important: registers all of the service providers configured for your application.

Simple service providers only have one method: `register`. This `register` method is called when the service provider is registered with the application object via the application's own `register` method. Within this method, service providers register things with the [IoC container](/docs/ioc). Essentially, each service provider binds one or more [closures](http://us3.php.net/manual/en/functions.anonymous.php) into the container, which allows you to access those bound services within your application. So, for example, the `QueueServiceProvider` registers closures that resolve the various [Queue](/docs/queues) related classes. Of course, service providers may be used for any bootstrapping task, not just registering things with the IoC container. A service provider may register event listeners, view composers, Artisan commands, and more.

After all of the service providers have been registered, your `app/start` files will be loaded. Lastly, your `app/routes.php` file will be loaded. Once your `routes.php` file has been loaded, the Request object is sent to the application so that it may be dispatched to a route.

So, let's summarize:

1. Request enters `public/index.php` file.
2. `bootstrap/start.php` file creates Application and detects environment.
3. Internal `framework/start.php` file configures settings and loads service providers.
4. Application `app/start` files are loaded.
5. Application `app/routes.php` file is loaded.
6. Request object sent to Application, which returns Response object.
7. Response object sent back to client.

Now that you have a good idea of how a request to a Laravel application is handled, let's take a closer look at "start" files!

<a name="start-files"></a>
## 启动文件

应用程序的启动文件被存放在`app/start`目录中。默认情况下，该目录下包含三个文件：`global.php`、`local.php` 和 `artisan.php`文件。需要获取更多关于`artisan.php`的信息，可以参考文档[Artisan 命令行](/docs/commands#registering-commands)。

`global.php`启动文件默认包含一些基本项目，例如[Logger](/docs/errors)的注册以及载入`app/filters.php` 文件。然而，你可以在该文件里做任何你想做的事情。无论在什么环境下，它都将会被自动包含进_每一个_request中。而`local.php` 文件仅在`local`环境下被执行。获取更多关于环境的信息，请查看文档[配置](/docs/configuration)。

当然，如果除了`local`环境你还有其他环境的话，你也可以为针对这些环境创建启动文件。这些文件将在应用程序运行在该环境中时被自动包含。So, for example, if you have a `development` environment configured in your `bootstrap/start.php` file, you may create a `app/start/development.php` file, which will be included when any requests enter the application in that environment.

### What To Place In Start Files

Start files serve as a simple place to place any "bootstrapping" code. For example, you could register a View composer, configure your logging preferences, set some PHP settings, etc. It's totally up to you. Of course, throwing all of your bootstrapping code into your start files can get messy. For large applications, or if you feel your start files are getting messy, consider moving some bootstrapping code into [service providers](/docs/ioc#service-providers).

<a name="application-events"></a>
## 应用程序事件

你还可以通过注册 `before`、`after`、`finish` 和 `shutdown`应用程序事件以便在处理request之前或后做一些操作：

**注册应用程序事件**

	App::before(function($request)
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

Listeners to these events will be run `before` and `after` each request to your application. These events can be helpful for global filtering or global modification of responses. You may register them in one of your `start` files or in a [service provider](/docs/ioc#service-providers).

The `finish` event is called after the response from your application has been sent back to the client. This is a good place to do any last minute processing your application requires. The `shutdown` event is called immediately after all of the `finish` event handlers finish processing, and is the last opportunity to do any work before the script terminates. Most likely, you will not have a need to use either of these events.

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)