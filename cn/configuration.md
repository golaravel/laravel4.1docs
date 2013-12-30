# 配置

- [简介](#introduction)
- [环境配置](#environment-configuration)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 简介

所有关于Laravel框架的配置文件都存放在`app/config`目录里。所有文件里的配置选项都有说明文档，因此你可以轻松的查看这些文件，并熟悉这些配置项。

当你需要在运行时访问配置项时，可以使用`Config`类：

**获取一个配置项的值**

	Config::get('app.timezone');

如果配置项不存在，你还可以指定返回的默认值：

	$timezone = Config::get('app.timezone', 'UTC');

注意"点"式语法可以用来访问不同文件里的配置项的值。你还可以在运行时为配置项赋值。:

**为配置项赋值**

	Config::set('database.default', 'sqlite');

在程序运行时设置的配置值只在本次请求中有效，不会对以后的请求造成影响。

<a name="environment-configuration"></a>
## 环境配置

通常应用程序根据不同的运行环境确定不同的配置项的值是非常有用的。例如，你也许希望在开发机器与生产机器上使用不同的缓存驱动（cache driver）。根据环境来改变配置就能很容易的达到此目的。

在`config`目录下创建一个和你的环境名同名的目录，例如`local`。然后，创建配置文件，这些配置文件中包含你想覆盖的配置选项。例如，要在本地环境中覆盖缓存驱动（cache driver），你可以在`app/config/local`目录中创建`cache.php`文件并包含如下内容：

    <?php
    
    return array(
    
    	'driver' => 'file',
    
    );

> **注意:** 不要使用'testing'作为环境名，它是专门为单元测试所保留的。

注意，你不需要为基础配置文件中的_所有_配置项指定值，只需指定你需要覆盖的配置选项即可。环境配置文件将会以"cascade"方式覆盖基本配置文件。

接下来，我们需要指导框架如何确定其运行环境。默认环境总是`produciton`。然而，你可以在安装目录的根目录下的`bootstrap/start.php` 文件中设置其他的环境。在该文件中，你可以找到 `$app->detectEnvironment`方法的调用。传入的数组参数就是用来确定当前运行环境的。你可以根据需要添加其他的环境或机器名。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

 在此案例中，'local' 是运行环境的名称，'your-machine-name' 是服务器的主机名。在Linux和Mac上，可以通过 `hostname` 命令来确定所用机器的主机名。

如果你需要更灵活的环境检查方式，可以在调用`detectEnvironment`时传递一个`闭包（Closure）` ， 这样你就可以按照自己的方式检查环境了：

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

可以通用调用`environment`方法来获取当前的应用环境：

**或当前的应用环境**

	$environment = App::environment();

你还可以将参数传递给 `environment` 方法来判断应用环境是否与某个给定值匹配：

	if (App::environment('local'))
	{
		// The environment is local
	}

	if (App::environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

<a name="maintenance-mode"></a>
## 维护模式

当应用处于维护模式时，所有的路由都会指向一个自定义的视图。这对于更新应用或执行维护任务时临时"禁用"当前应用是很方便的。`App::down`方法在`app/start/global.php`文件里进行了定义，它将在维护模式时将该方法输出的内容展示给用户。  

要开启维护模式，只需执行Artisan 的 `down`命令：

	php artisan down

要关闭维护模式，只需执行 `up` 命令：

	php artisan up

当你的应用处于维护模式时，如需展示一个自定义的视图，只要在`app/start/global.php`文件中加入如下代码即可：

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});


如果传递给 `down` 方法的闭包返回值为 `NULL`, 在此次请求中将忽略维护模式。

### 维护模式 & 队列

当应用程序处于维护模式时, 将不接受新的[队列任务](/docs/queues) 。一旦应用程序退出维护模式，队列任务的处理即恢复正常。

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)