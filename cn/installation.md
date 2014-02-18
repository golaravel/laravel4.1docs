# 安装

- [安装Composer](#install-composer)
- [安装Laravel](#install-laravel)
- [服务器环境要求](#server-requirements)
- [配置](#configuration)
- [优雅链接](#pretty-urls)

<a name="install-composer"></a>
## 安装Composer

Laravel框架使用[Composer](http://getcomposer.org)（PHP包管理工具，参考 [Composer 中文文档](http://composer.golaravel.com/)）来管理代码依赖性。
首先，你需要下载Composer的PHAR打包文件（ `composer.phar` ），下载完成后把它放在项目目录下或者放到 `usr/local/bin` 目录下以便在系统中全局调用。在Windows操作系统中，你可以使用Composer的[Windows安装工具](https://getcomposer.org/Composer-Setup.exe)。

<a name="install-laravel"></a>
## 安装Laravel

### 通过 Laravel 安装器安装

先下载 [Laravel 安装器 PHAR 文件](http://laravel.com/laravel.phar)。为了方便起见，将该文件重命名为 `laravel` 并移动至 `/usr/local/bin` 目录。完成后，只需在你指定的目录下输入简单的 `laravel new` 命令即可创建一个全新的Laravel安装。例如，`laravel new blog` 将创建一个名为 `blog` 的目录，该目录中包含了一份全新的Laravel安装以及所需的依赖项。这种安装方法比通过 Composer 安装要快许多。

### 通过 Composer create-project 命令安装Laravel

还可以通过在命令行执行 Composer `create-project` 命令来安装Laravel：

	composer create-project laravel/laravel --prefer-dist

### 通过下载Laravel包安装

Composer安装完成后，下载[最新版Laravel框架](https://github.com/laravel/laravel/archive/master.zip)，把它解压缩到你服务器上的一个目录中。然后在Laravel应用的根目录下运行命令行命令 `php composer.phar install` （或者 `composer install` ）来安装所有的框架依赖包。在此过程中，为了成功完成安装，你需要在服务器上安装好Git。

当Laravel框架安装好后，你可以使用命令行命令 `php composer.phar update` 来更新框架。

<a name="server-requirements"></a>
## 服务器环境要求

Laravel框架有一些系统要求：

- PHP最低版本： 5.3.7
- MCrypt PHP扩展

从PHP 5.5版本开始，针对某些操作系统的安装包需要你自己手工安装PHP的JSON扩展模块。如果你使用的是Ubuntu，可以通过,  `apt-get install php5-json` 命令直接安装。（译者注：还是Ubuntu傻瓜化啊！！！）

<a name="configuration"></a>
## 配置

Laravel框架几乎无需配置就可立即使用。你可以自由地快速开始开发。然而，你也许希望先查看下 `app/config/app.php` 配置文件和相关的文档说明。它包含了一些你也许要修改的配置选项，如 `时区` 和 `地区` 等。

<a name="permissions"></a>
### 权限设置
Laravel框架有一个目录需要额外设置权限： 需要为 app/storage 目录下的文件设置写权限。

<a name="paths"></a>
### 路径设置

一些框架目录路径是可以设置的。如果需要改变这些目录的位置，可以查看 `bootstrap/paths.php` 文件中的设置。

<a name="pretty-urls"></a>
## 优雅链接

Laravel框架通过设置 `public/.htaccess` 文件去除链接中的`index.php`。 如果你你的服务器使用的是Apache，请开启`mod_rewrite` 模块。

如果框架附带的 `.htaccess` 文件在你的Apache环境中不起作用，请尝试下面这个版本：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^(.+)/$ http://%{HTTP_HOST}/$1 [R=301,L]

	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]
