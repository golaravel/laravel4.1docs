# SSH(Secure Shell)

- [配置](#configuration)
- [基本用法](#basic-usage)
- [任务](#tasks)
- [SFTP上传](#sftp-uploads)
- [显示远程日志的末尾几行](#tailing-remote-logs)

<a name="configuration"></a>
## 配置

Laravel 为 SSH 登录远程服务器并运行命令提供了一种简单的方式，允许你轻松创建运行在远程服务器上的 Artisan 任务。`SSH` 门面类(facade)提供了连接远程服务器和运行命令的访问入口。

配置文件位于 `app/config/remote.php`，该文件包含了配置远程连接所需的所有选项。`connections` 数组包含了一个以名称作为键的服务器列表。只需将 `connections` 数组中的凭证信息填好，就可以开始运行远程任务了。注意 `SSH` 可以使用密码或 SSH 密钥进行身份验证。

<a name="basic-usage"></a>
## 基本用法

**在默认服务器上运行命令**

使用 `SSH::run` 方法在 `default` 远程服务器连接上运行命令：

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

**在指定连接上运行命令**

或者，你可以使用 `into` 方法在指定的服务器连接上运行命令：

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

**捕获命令输出结果**

你可以通过向 `run` 方法传递一个闭包来捕获远程命令的“实时”输出结果：

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## 任务
<a name="tasks"></a>

如果需要定义一组经常放在一起运行的命令，你可以使用 `define` 方法来定义一个任务( `task` )：

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

任务一旦创建，就可以使用 `task` 来执行：

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-uploads"></a>
## SFTP 上传

`SSH` 类的 `put` 和 `putString` 方法提供了一种简单的上传方式，用于将文件或字符串上传到服务器：

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString('Foo', $remotePath);

<a name="tailing-remote-logs"></a>
## 显示远程日志的末尾几行

Laravel 提供了一个有用的命令用来查看任何远程连接服务器上的 `laravel.log` 文件的末尾几行内容。只需使用 Artisan 命令 `tail` 并指定你想要查看日志的远程连接的名称即可：

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file