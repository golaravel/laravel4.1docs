# 错误 & 日志

- [错误详情](#error-detail)
- [处理错误](#handling-errors)
- [HTTP异常](#http-exceptions)
- [处理404错误](#handling-404-errors)
- [日志](#logging)

<a name="error-detail"></a>
## 错误详情

默认情况下，应用中会默认输出错误详情。这就是说当有错误发生时，你将看到一个详细的堆栈轨迹（stack trace）和错误信息页面。你可以在`app/config/app.php`文件中将`debug`配置选项设为`false`来关闭它。

> **Note:** It is strongly recommended that you turn off error detail in a production environment.

<a name="handling-errors"></a>
## 处理错误

默认情况下，`app/start/global.php`文件中包含了一个用于处理所有异常的处理器：

	App::error(function(Exception $exception)
	{
		Log::error($exception);
	});

这是一个最基本错误处理器。当然，你也可以自己定义更多的错误处理器。框架会根据你在定义错误处理器时指定的异常类型（type-hint）来分别调用。例如，你定义一个仅处理`RuntimeException`异常的处理器：

	App::error(function(RuntimeException $exception)
	{
		// Handle the exception...
	});

如果异常处理器返回一个response，此response将会发送给浏览器，其他异常处理器将不会再被调用：

	App::error(function(InvalidUserException $exception)
	{
		Log::error($exception);

		return 'Sorry! Something is wrong with this account!';
	});

若要监听PHP中的致命错误（fatal error），可以使用`App::fatal`方法：

	App::fatal(function($exception)
	{
		//
	});

如果同时有多个异常处理器，应该先定义最通用的，然后定义最具体的异常处理器。例如，处理所有 `Exception` 类型的异常处理器应当在处理 `Illuminate\Encryption\DecryptException` 类型的异常处理器之前。

### Where To Place Error Handlers

There is no default "home" for error handler registrations. Laravel offers you freedom in this area. One option is to define the handlers in your `start/global.php` file. In general, this is a convenient location to place any "bootstrapping" code. If that file is getting crowded, you could create an `app/errors.php` file, and `require` that file from your `start/global.php` script. A third option is to create a [service provider](/docs/ioc#service-providers) that registers the handlers. Again, there is no single "correct" answer. Choose a location that you are comfortable with.

<a name="http-exceptions"></a>
## HTTP 异常

Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404), an "unauthorized error" (401) or even a developer generated 500 error. In order to return such a response, use the following:

	App::abort(404);

Optionally, you may provide a response:

	App::abort(403, 'Unauthorized action.');

This method may be used at any time during the request's lifecycle.

<a name="handling-404-errors"></a>
## 处理404错误

你可以注册一个错误处理器来处理应用中的所有"404 Not Found"错误，这样就可以更早的返回一个自定义的404错误页面：

	App::missing(function($exception)
	{
		return Response::view('errors.missing', array(), 404);
	});

<a name="logging"></a>
## 日志

Laravel日志工具构建在强大的[Monolog](http://github.com/seldaek/monolog)库之上，并提供了一层简单的抽象层。默认情况下，Laravel为应用创建一个单独的日志文件，此文件的路径是`app/storage/laravel.log`。你可以通过如下方法输出log：

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');

日志工具提供了7个日志级别，参见[RFC 5424](http://tools.ietf.org/html/rfc5424)：**debug**、**info**、**notice**、**warning**、**error**、**critical** 和 **alert**。

还可以像Log方法传递一个包含额外数据的数组：

	Log::info('Log message', array('context' => 'Other helpful information'));

Monolog提供了多种处理日志的功能。如果需要，你可以通过如下方法获取Laravel内部所使用的Monolog实例：

	$monolog = Log::getMonolog();

你还可以注册一个事件，用以获取所有传输到日志的信息：

**注册一个日志监听器**

	Log::listen(function($level, $message, $context)
	{
		//
	});

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)
