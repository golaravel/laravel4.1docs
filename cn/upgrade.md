# 升级向导

- [Upgrading To 4.1.29 From <= 4.1.x](#upgrade-4.1.29)
- [Upgrading To 4.1.26 From <= 4.1.25](#upgrade-4.1.26)
- [从 4.0 升级到 4.1](#upgrade-4.1)

<a name="upgrade-4.1.29"></a>
## Upgrading To 4.1.29 From <= 4.1.x

Laravel 4.1.29 improves the column quoting for all database drivers. This protects your application from some mass assignment vulnerabilities when **not** using the `fillable` property on models. If you are using the `fillable` property on your models to protect against mass assignemnt, your application is not vulerable. However, if you are using `guarded` and are passing a user controlled array into an "update" or "save" type function, you should upgrade to `4.1.29` immediately as your application may be at risk of mass assignment.

To upgrade to Laravel 4.1.29, simply `composer update`. No breaking changes are introduced in this release.

<a name="upgrade-4.1.26"></a>
## Upgrading To 4.1.26 From <= 4.1.25

Laravel 4.1.26 introduces security improvements for "remember me" cookies. Before this update, if a remember cookie was hijacked by another malicious user, the cookie would remain valid for a long period of time, even after the true owner of the account reset their password, logged out, etc.

This change requires the addition of a new `remember_token` column to your `users` (or equivalent) database table. After this change, a fresh token will be assigned to the user each time they login to your application. The token will also be refreshed when the user logs out of the application. The implications of this change are: if a "remember me" cookie is hijacked, simply logging out of the application will invalidate the cookie.

### Upgrade Path

First, add a new, nullable `remember_token` of VARCHAR(100), TEXT, or equivalent to your `users` table.

Next, if you are using the Eloquent authentication driver, update your `User` class with the following three methods:

	public function getRememberToken()
	{
		return $this->remember_token;
	}

	public function setRememberToken($value)
	{
		$this->remember_token = $value;
	}

	public function getRememberTokenName()
	{
		return 'remember_token';
	}

> **Note:** All existing "remember me" sessions will be invalidated by this change, so all users will be forced to re-authenticate with your application.

### Package Maintainers

Two new methods were added to the `Illuminate\Auth\UserProviderInterface` interface. Sample implementations may be found in the default drivers:

	public function retrieveByToken($identifier, $token);

	public function updateRememberToken(UserInterface $user, $token);

The `Illuminate\Auth\UserInterface` also received the three new methods described in the "Upgrade Path".

<a name="upgrade-4.1"></a>
## 从 4.0 升级到 4.1

### 升级 Composer 依赖项

为把应用程序升级为 Laravel 4.1，需要将 `composer.json` 文件中 `laravel/framework` 的版本改为 `4.1.*` 。

### 替换文件

把 `public/index.php` 文件替换为 [仓库里这个新版本](https://github.com/laravel/laravel/blob/master/public/index.php)。

把 `artisan` 文件替换为 [仓库里这个新版本](https://github.com/laravel/laravel/blob/master/artisan)。

### 添加配置文件 & 选项

更新 `app/config/app.php` 配置文件中的 `aliases` 和 `providers` 数组。更新后的数组值可以 [在这个文件中](https://github.com/laravel/laravel/blob/master/app/config/app.php) 找到。记得在更新完成后将你自定义的服务提供器/别名添加回数组中。

[从仓库](https://github.com/laravel/laravel/blob/master/app/config/remote.php) 添加新的 `app/config/remote.php` 文件。

在 `app/config/session.php` 文件中添加新的 `expire_on_close` 配置选项，默认值为 `false`。

在 `app/config/queue.php` 文件中添加新的 `failed` 配置区，默认值如下：

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(可选)** 更新 `app/config/view.php` 文件中的 `pagination` 配置选项为 `pagination::slider-3`。

### 更新控制器

如果 `app/controllers/BaseController.php` 的顶部有 `use` 语句，把 `use Illuminate\Routing\Controllers\Controller;` 改为 `use Illuminate\Routing\Controller;`。

### 更新密码提示功能

框架的密码提示功能进行了较大修改以提供更好的灵活性。你可以运行 Artisan 命令 `php artisan auth:reminders-controller` 来查看新版密码提示控制器是什么样的。你还可以浏览并根据 [更新过的文档](/docs/security#password-reminders-and-reset) 来更新你的应用程序。

更新 `app/lang/en/reminders.php` 语言文件以匹配 [这个更新过的文件](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php)。

### 更新环境检测

出于安全原因，不再使用 URL 域名来检测你的应用程序环境，因为很容易被攻击者伪造从而修改请求的运行环境。你应该改用主机名(Mac、Linux 和 Windows 的 `hostname` 命令)来进行环境检测。

### 简化的日志文件

Laravel 现在创建一个单一的日志文件：`app/storage/logs/laravel.log`。不过，你还是可以在 `app/start/global.php` 文件中配置日志行为。

### 移除重定向的尾部斜杠

在 `bootstrap/start.php` 文件中，移除 `$app->redirectIfTrailingSlash()` 调用。由于该方法的功能已经通过框架自带的 `.htaccess` 文件实现了，所以不再需要了。

然后，替换 Apache 的 `.htaccess` 文件为 [这个新版本](https://github.com/laravel/laravel/blob/master/public/.htaccess)，用于处理尾部斜杠。

### 获取当前路由

现在通过 `Route::current()` 获取当前路由，以前是 `Route::getCurrentRoute()`。

### Composer 更新

完成上述修改后，运行 `composer update` 更新应用程序的核心文件即可。If you receive class load errors, try running the `update` command with the `--no-scripts` option enabled like so: `composer update --no-scripts`.

### Wildcard Event Listeners

The wildcard event listeners no longer append the event to your handler functions parameters. If you require finding the event that was fired you should use `Event::firing()`.

