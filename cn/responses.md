# 视图 & Response

- [基本Response](#basic-responses)
- [重定向](#redirects)
- [视图](#views)
- [视图合成](#view-composers)
- [特殊Response](#special-responses)
- [Response Macros](#response-macros)

<a name="basic-responses"></a>
## 基本Response

**从路由中返回字符串**

	Route::get('/', function()
	{
		return 'Hello World';
	});

**创建自定义Response**

`Response`类继承自`Symfony\Component\HttpFoundation\Response`类，提供了多种方法用于构建HTTP Response。

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

If you need access to the `Response` class methods, but want to return a view as the response content, you may use the `Response::view` method for convenience:

	return Response::view('hello')->header('Content-Type', $type);

**在Response中添加Cookie**

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## 重定向

**返回一个重定向**

	return Redirect::to('user/login');

**Returning A Redirect With Flash Data**

	return Redirect::to('user/login')->with('message', 'Login Failed');

> **Note:** Since the `with` method flashes data to the session, you may retrieve the data using the typical `Session::get` method.

**返回一个重定向至命名路由**

	return Redirect::route('login');

**返回一个重定向至带有参数的命名路由**

	return Redirect::route('profile', array(1));

**返回一个重定向至带有命名参数的命名路由**

	return Redirect::route('profile', array('user' => 1));

**返回一个重定向至控制器Action**

	return Redirect::action('HomeController@index');

**返回一个重定向至控制器Action并带有参数**

	return Redirect::action('UserController@profile', array(1));

**返回一个重定向至控制器Action并带有命名参数**

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## 视图

视图通常包含应用中的HTML代码，为分离表现层与控制器和业务逻辑提供了便利。视图存放于`app/views`目录。

一个简单视图案例：

	<!-- View stored in app/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

通过如下方法来返回该视图到浏览器：

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

传递给`View::make`方法的第二个参数是一个数组，它将被传递给视图。

**传递数据给视图**

	// Using conventional approach
	$view = View::make('greeting')->with('name', 'Steve');

	// Using Magic Methods
	$view = View::make('greeting')->withName('steve');

在上面的案例中，`$name`变量在视图内是可以访问的，其值为`Steve`。

你还可以在所有视图同共享同一数据：

	View::share('name', 'Steve');

**向视图传递子视图**

或许你可能想将一个视图放入到另一个视图中。例如，将存放在`app/views/child/view.php`文件中的子视图传递给另一视图，如下：

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

在父视图就可以输出该子视图了：

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## 视图合成器

视图合成器可以是回调函数或者类方法，它们在创建视图时被调用。如果你想在应用程序中，每次创建视图时都为其绑定一些数据，使用视图合成器可以将代码组织到一个地方。因此，视图合成器就好像是 “视图模型”或者是“主持人”。

**定义一个视图合成器**

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

现在，每次创建`profile`视图时，`count`都会被绑定到视图中。

你也可以为多个视图同时绑定一个视图合成器：

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

如果你更喜欢使用基于类的视图合成器，[IoC container](/docs/ioc)可以提供更多便利，如下所示：

	View::composer('profile', 'ProfileComposer');

视图合成器类定义如下：

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

注意，没有规定视图合成器类存放在哪里。因此，你可以任意存放，只要能在`composer.json`文件中指定位置并自动加载即可。

### View Creators

View **creators** work almost exactly like view composers; however, they are fired immediately when the view is instantiated. To register a view creator, simple use the `creator` method:

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## 特殊Response

**创建一个JSON Response**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

**创建一个JSONP Response**

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

**创建一个文件下载Response**

	return Response::download($pathToFile);

	return Response::download($pathToFile, $status, $headers);

> **Note:** Symfony HttpFoundation, which manages file downloads, requires the file being downloaded to have an ASCII file name.

<a name="response-macros"></a>
## Response Macros

If you would like to define a custom response that you can re-use in a variety of your routes and controllers, you may use the `Response::macro` method:

	Response::macro('caps', function($value)
	{
		return Response::make(strtoupper($value));
	});

The `macro` function accepts a name as its first argument, and a Closure as its second. The macro's Closure will be executed when calling the macro name on the `Response` class:

	return Response::caps('foo');

You may define your macros in one of your `app/start` files. Alternatively, you may organize your macros into a separate file which is included from one of your `start` files.

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)
