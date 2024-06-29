<!--yml

类别：未分类

日期：2024-05-27 12:55:20

-->

# PHP: The Right Way

> 来源：[https://phptherightway.com/](https://phptherightway.com/)

# 欢迎

网络上有大量过时的信息会误导新的 PHP 用户，传播不良实践和不安全的代码。*PHP: The Right Way* 是一个易于阅读的快速参考，符合 PHP 流行的编码标准，提供链接到权威教程的内容，并反映了当前被视为最佳实践的贡献者的看法。

*没有一种官方的 PHP 使用方式*。本网站旨在向新的 PHP 开发人员介绍一些可能会在为时已晚之前未发现的主题，并旨在为有经验的专业人士提供一些长期从事但从未重新考虑的新思路。本网站也不会告诉你使用哪些工具，而是在可能时为多种选项提供建议，并解释方法和用例之间的区别。

这是一份活跃的文档，将继续更新更多有用的信息和示例。

## 翻译

*PHP: The Right Way* 已被翻译成许多不同的语言：

## 书籍

*PHP: The Right Way* 的最新版本也以 PDF、EPUB 和 MOBI 格式提供。 [前往 Leanpub](https://leanpub.com/phptherightway)

## 如何贡献

帮助使这个网站成为新 PHP 程序员的最佳资源！ [在 GitHub 上贡献](https://github.com/codeguy/php-the-right-way/tree/gh-pages)

## 使用当前稳定版本（8.3）

如果你刚开始使用 PHP，请从当前稳定版本 [PHP 8.3](https://www.php.net/downloads.php) 开始。PHP 8.x 在旧的 7.x 和 5.x 版本之上增加了许多 [新功能](#language_highlights)。引擎已经大规模重写，PHP 现在比旧版本更快。PHP 8 是语言的一个重大更新，包含许多新功能和优化。

你应该尽快升级到最新的稳定版本 - PHP 7.4 [已经终止生命周期](https://www.php.net/supported-versions.php)。升级很容易，因为没有太多向后兼容性断裂 [PHP 8.0](https://www.php.net/manual/migration80.incompatible.php), [PHP 8.1](https://www.php.net/manual/migration81.incompatible.php), [PHP 8.2](https://www.php.net/manual/migration82.incompatible.php), [PHP 8.3](https://www.php.net/manual/migration83.incompatible.php)。如果你不确定一个函数或特性在哪个版本中可用，可以在 [php.net](https://www.php.net/manual/) 网站上查看 PHP 文档。

## 内置的 web 服务器

使用 PHP 5.4 或更新版本，你可以在不安装和配置完整的 Web 服务器的情况下开始学习 PHP。要启动服务器，请在项目的 Web 根目录中的终端中运行以下命令：

```
>  php -S localhost:8000
```

## macOS 设置

macOS 预装了 PHP，但通常会滞后于最新的稳定发布版。在 macOS 上安装最新的 PHP 版本有多种方法。

### 通过 Homebrew 安装 PHP

[Homebrew](https://brew.sh/)是macOS的软件包管理器，帮助您轻松安装PHP和各种扩展。Homebrew核心存储库为PHP 7.4、8.0、8.1、8.2和PHP 8.3提供了“配方”。使用此命令安装最新版本：

你可以通过修改你的`PATH`变量在Homebrew PHP版本之间切换。或者，你可以使用[brew-php-switcher](https://github.com/philcook/brew-php-switcher)自动切换PHP版本。

您还可以通过取消链接并链接所需版本手动切换PHP版本：

```
brew unlink php
brew link --overwrite php@8.2 
```

```
brew unlink php
brew link --overwrite php@8.3 
```

### 通过Macports安装PHP

[MacPorts](https://www.macports.org/install.php)项目是一个开源社区倡议，旨在设计一个易于使用的系统，用于在macOS操作系统上编译、安装和升级基于命令行、X11或Aqua的开源软件。

MacPorts支持预编译的二进制文件，因此您无需从源tarball文件重新编译每个依赖项，如果您的系统上没有安装任何软件包，它会挽救您的生命。

在此时，您可以使用`port install`命令安装`php54`、`php55`、`php56`、`php70`、`php71`、`php72`、`php73`、`php74`、`php80`、`php81`、`php82`或`php83`，例如：

```
sudo port install php74
sudo port install php83 
```

并且你可以运行`select`命令来切换你的活动PHP：

```
sudo port select --set php php83 
```

### 通过phpbrew安装PHP

[phpbrew](https://github.com/phpbrew/phpbrew)是一个用于安装和管理多个PHP版本的工具。如果两个不同的应用程序/项目需要不同版本的PHP，并且您没有使用虚拟机，这将非常有用。

### 通过Liip的二进制安装程序安装PHP

另一个流行的选项是[php-osx.liip.ch](https://web.archive.org/web/20220505163210/https://php-osx.liip.ch/)，它提供了逐行安装方法，适用于5.3到7.3版本。它不会覆盖Apple安装的PHP二进制文件，而是将所有内容安装在单独的位置（/usr/local/php5）。

### 从源代码编译

另一个选项是自行[编译安装](https://www.php.net/install.macosx.compile)PHP，这种情况下，请确保已安装[Xcode](https://github.com/kennethreitz/osx-gcc-installer)或苹果的替代品“XCode的命令行工具”，可从苹果的开发者中心下载。

### All-in-One安装程序

上述解决方案主要处理PHP本身，并不提供像[Apache](https://httpd.apache.org/)、[Nginx](https://www.nginx.com/)或SQL服务器等内容。“全合一”解决方案如[MAMP](https://www.mamp.info/en/downloads/)和[XAMPP](https://www.apachefriends.org/)将为您安装这些其他软件并将它们整合在一起，但设置的简便性会伴随着灵活性的牺牲。

## Windows设置

你可以从[windows.php.net/download](https://windows.php.net/download/)下载二进制文件。在解压PHP后，建议将[PATH](https://www.windows-commandline.com/set-path-command-line/)设置为PHP文件夹的根目录（即php.exe所在的位置），这样你就可以在任何地方执行PHP了。

对于学习和本地开发，你可以使用内置的PHP 5.4+ Web服务器，无需担心配置问题。如果你需要一个“一站式”解决方案，包括完整的Web服务器和MySQL，那么像[XAMPP](https://www.apachefriends.org/)、[EasyPHP](https://www.easyphp.org/)、[OpenServer](https://ospanel.io/)和[WAMP](https://www.wampserver.com/en/)这样的工具将帮助你快速搭建Windows开发环境。尽管如此，这些工具与生产环境可能有些不同，如果你在Windows上开发并部署到Linux，务必注意环境差异。

如果你需要在Windows上运行生产系统，那么IIS7将为你提供最稳定和最佳的性能。你可以使用[phpmanager](http://phpmanager.codeplex.com/)（一个IIS7的GUI插件）来简化PHP的配置和管理。IIS7自带FastCGI，并已准备就绪，你只需将PHP配置为处理程序。关于PHP的支持和额外资源，请访问[iis.net上的专门区域](https://php.iis.net/)。

通常，在开发和生产环境中运行应用程序可能会导致一些奇怪的Bug在上线时冒出来。如果你在Windows上开发并部署到Linux（或其他非Windows环境），那么你应该考虑使用[虚拟机](/#virtualization_title)。

Chris Tankersley在他的博客文章中详细介绍了他使用Windows进行[PHP开发的工具](https://ctankersley.com/2016/11/13/developing-on-windows-2016/)。

## Linux设置

大多数GNU/Linux发行版都提供了来自官方仓库的PHP，但这些包通常稍微落后于当前稳定版本。在这些发行版上，获取更新的PHP版本有多种方法。例如，在Ubuntu和基于Debian的GNU/Linux发行版上，最佳选择是由[Ondřej Surý](https://deb.sury.org/)提供和维护的个人软件包存档（PPA）在Ubuntu上，以及Debian的DPA/bikeshed。以下是每种方法的详细说明。总之，你始终可以使用容器、编译PHP源代码等方法。

### 基于Ubuntu的发行版

对于Ubuntu发行版，[Ondřej Surý的PPA](https://launchpad.net/~ondrej/+archive/ubuntu/php)提供了支持的PHP版本以及许多PECL扩展。要将此PPA添加到你的系统中，请在终端中执行以下步骤：

1.  首先，使用以下命令将PPA添加到你系统的软件源：

    ```
    sudo add-apt-repository ppa:ondrej/php 
    ```

1.  在添加PPA后，更新你系统的软件包列表：

这将确保你的系统可以访问并安装PPA中提供的最新PHP包。

#### 基于 Debian 的发行版

对于基于 Debian 的发行版，Ondřej Surý 还提供了一个 [bikeshed](https://packages.sury.org/php/)（类似于 PPA 的 Debian 仓库）。要将 bikeshed 添加到您的系统并更新它，请按照以下步骤进行：

1.  确保您具有 root 访问权限。如果没有，请在以下命令中使用 `sudo`。

1.  更新系统的软件包列表：

1.  安装 `lsb-release`、`ca-certificates` 和 `curl`：

    ```
    sudo apt-get -y install lsb-release ca-certificates curl 
    ```

1.  下载仓库的签名密钥：

    ```
    sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg 
    ```

1.  将仓库添加到系统的软件源：

    ```
    sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list' 
    ```

1.  最后，再次更新系统的软件包列表：

执行这些步骤后，您的系统将能够从 bikeshed 安装最新的 PHP 软件包。

## 常见的目录结构

在开始编写网页程序时，一个常见的问题是：“我把我的东西放哪里？”多年来，这个答案一直是“`DocumentRoot` 所在的地方”。虽然这个答案不完整，但它是一个很好的起点。

出于安全原因，配置文件不应该被站点的访客访问；因此，公共脚本存放在公共目录中，而私有配置和数据则存放在该目录之外。

每个团队、CMS 或框架都使用标准的目录结构。然而，如果一个人独立开始一个项目，知道要使用哪种文件系统结构可能令人望而生畏。

[Paul M. Jones](https://paul-m-jones.com/) 对 PHP 领域数万个 GitHub 项目的常见实践进行了一些出色的研究。他基于这些研究编制了一个标准的文件和目录结构，即 [Standard PHP Package Skeleton](https://github.com/php-pds/skeleton)。在这个目录结构中，`DocumentRoot` 应指向 `public/`，单元测试应位于 `tests/` 目录中，由 [composer](/#composer_and_packagist) 安装的第三方库应放在 `vendor/` 目录中。对于其他文件和目录，遵循 [Standard PHP Package Skeleton](https://github.com/php-pds/skeleton) 对项目贡献者来说是最合理的。

# 代码风格指南

PHP 社区庞大多样，由无数库、框架和组件组成。PHP 开发者通常会选择几个这些组件，并将它们结合到一个项目中。重要的是，PHP 代码尽可能遵循共同的代码风格，以便开发者可以轻松地混合和匹配各种库用于他们的项目。

[Framework Interop Group](https://www.php-fig.org/)提出并批准了一系列风格建议。并非所有建议都涉及代码风格，但其中涉及的有[PSR-1](https://www.php-fig.org/psr/psr-1/)、[PSR-12](https://www.php-fig.org/psr/psr-12/)、[PSR-4](https://www.php-fig.org/psr/psr-4/)和[PER编码风格](https://www.php-fig.org/per/coding-style/)。这些建议只是一套规则，许多项目（如Drupal、Zend、Symfony、Laravel、CakePHP、phpBB、AWS SDK、FuelPHP、Lithium等）正在采纳。您可以为自己的项目使用它们，或者继续使用您自己的个人风格。

理想情况下，您应该编写符合已知标准的PHP代码。这可以是任何PSR的组合，或者由PEAR或Zend制定的编码标准之一。这意味着其他开发人员可以轻松阅读和使用您的代码，并且实现这些组件的应用程序即使在处理大量第三方代码时也能保持一致性。

您可以使用[PHP_CodeSniffer](https://github.com/PHPCSStandards/PHP_CodeSniffer)来检查代码是否符合这些建议之一，并且像[Sublime Text](https://github.com/benmatselby/sublime-phpcs)这样的文本编辑器插件可以提供实时反馈。

您可以通过以下工具之一自动修复代码布局：

您也可以从shell手动运行phpcs：

```
phpcs -sw --standard=PSR1 file.php 
```

它将显示错误并描述如何修复它们。在git的预提交钩子中包含`phpcs`命令与`--filter=GitStaged` CLI参数也可能有帮助。这样，包含违反所选标准的代码将不能进入代码库，直到这些违规被修复。

如果你已经安装了PHP_CodeSniffer，那么你可以使用[PHP代码美化和修复工具](https://github.com/PHPCSStandards/PHP_CodeSniffer/wiki/Fixing-Errors-Automatically)自动修复它报告的代码布局问题。

```
phpcbf -w --standard=PSR1 file.php 
```

另一个选择是使用[PHP Coding Standards Fixer](https://cs.symfony.com/)。它会显示在修复错误之前代码结构存在什么样的错误。

```
php-cs-fixer fix -v --rules=@PSR1 file.php 
```

所有符号名称和代码基础都应使用英语。注释可以用任何语言编写，只要所有当前和未来可能在代码库上工作的人都可以轻松阅读。

最后，编写清洁的PHP代码的一个很好的补充资源是[Clean Code PHP](https://github.com/jupeter/clean-code-php)。

## 编程范式

PHP是一种灵活的、动态的语言，支持多种编程技术。多年来它发生了巨大的变化，特别是在PHP 5.0（2004年）中添加了坚实的面向对象模型，PHP 5.3（2009年）中添加了匿名函数和命名空间，以及PHP 5.4（2012年）中添加了traits。

### 面向对象编程

PHP拥有非常完整的面向对象编程功能，包括支持类、抽象类、接口、继承、构造函数、克隆、异常等。

### 函数式编程

PHP 支持一级函数，意味着函数可以分配给变量。用户定义的和内置函数都可以由变量引用并动态调用。函数可以作为参数传递给其他函数（称为*高阶函数*的功能），并且函数可以返回其他函数。

递归是一种允许函数调用自身的特性，语言本身支持，但大多数 PHP 代码更注重迭代。

PHP 5.3（2009年）以来引入了支持闭包的新匿名函数。

PHP 5.4 添加了将闭包绑定到对象作用域的能力，并且改进了可调用对象的支持，使它们几乎可以在所有情况下与匿名函数可互换使用。

PHP 通过反射 API 和魔术方法等机制支持各种形式的元编程。有许多可用的魔术方法，如 `__get()`、`__set()`、`__clone()`、`__toString()`、`__invoke()` 等，允许开发者钩入类的行为。Ruby 开发者经常说 PHP 缺少 `method_missing`，但作为 `__call()` 和 `__callStatic()` 存在。

## 命名空间

如上所述，PHP 社区有许多开发者创建大量代码。这意味着一个库的 PHP 代码可能与另一个库使用相同的类名。当这两个库在同一个命名空间中使用时，它们会发生冲突并引起问题。

*命名空间* 解决了这个问题。如 PHP 参考手册中所述，命名空间可类比于操作系统目录，它可以*命名空间*文件；两个同名文件可以存在于不同目录中。同样，两个同名的 PHP 类可以存在于不同的 PHP 命名空间中。就是这么简单。

编写代码时，为其添加命名空间是非常重要的，这样可以确保其他开发者在使用时不会与其他库发生冲突。

使用命名空间的推荐方法之一详见 [PSR-4](https://www.php-fig.org/psr/psr-4/)，旨在提供标准的文件、类和命名空间约定，以支持即插即用的代码。

2014年10月，PHP-FIG 弃用了之前的自动加载标准：[PSR-0](https://www.php-fig.org/psr/psr-0/)。PSR-0 和 PSR-4 都仍然可以完美使用。后者需要 PHP 5.3，因此许多仅支持 PHP 5.2 的项目实现了 PSR-0。

如果您要为新应用程序或软件包使用自动加载器标准，请查看 PSR-4。

## 标准 PHP 库

标准 PHP 库（SPL）随 PHP 打包提供了一组类和接口。主要由常用的数据结构类（堆栈、队列、堆等）和可以遍历这些数据结构或实现 SPL 接口的自定义类组成。

## 命令行界面

PHP 最初用于编写 Web 应用程序，但也非常适用于脚本化命令行界面（CLI）程序。命令行 PHP 程序可以帮助自动化诸如测试、部署和应用程序管理等常见任务。

CLI PHP程序非常强大，因为您可以直接使用应用程序的代码，而无需为其创建和保护Web GUI。只需确保**不要**将CLI PHP脚本放在公共Web根目录下！

尝试从命令行运行PHP：

```
>  php -i
```

`-i`选项将像[`phpinfo()`](https://www.php.net/function.phpinfo)函数一样打印您的PHP配置。

`-a`选项提供一个交互式shell，类似于ruby的IRB或python的交互式shell。还有许多其他有用的[命令行选项](https://www.php.net/features.commandline.options)。

让我们编写一个简单的“Hello, $name”CLI程序。要尝试它，请创建名为`hello.php`的文件，如下所示。

```
<?php
if ($argc !== 2) {
    echo "Usage: php hello.php <name>" . PHP_EOL;
    exit(1);
}
$name = $argv[1];
echo "Hello, $name" . PHP_EOL;
```

PHP根据脚本运行的参数设置了两个特殊变量。[`$argc`](https://www.php.net/reserved.variables.argc)是一个整数变量，包含参数*计数*，而[`$argv`](https://www.php.net/reserved.variables.argv)是一个数组变量，包含每个参数的*值*。第一个参数始终是您的PHP脚本文件的名称，在这种情况下是`hello.php`。

`exit()`表达式与非零数字一起使用，通知shell命令执行失败。常用的退出代码可以在[这里](https://www.gsp.com/cgi-bin/man.cgi?section=3&topic=sysexits)找到。

要从命令行运行我们的脚本，请参考上述：

```
>  php hello.php
Usage: php hello.php <name>  >  php hello.php world
Hello, world
```

## Xdebug

在软件开发中最有用的工具之一是合适的调试器。它允许您跟踪代码的执行并监视堆栈的内容。Xdebug，PHP的调试器，可以通过各种IDE使用提供断点和堆栈检查。它还可以允许像PHPUnit和KCacheGrind这样的工具执行代码覆盖分析和代码分析。

如果您发现自己陷入困境，愿意使用`var_dump()`/`print_r()`，但仍然找不到解决方案 - 或许您需要使用调试器。

[安装Xdebug](https://xdebug.org/docs/install)可能有些棘手，但它最重要的特性之一是“远程调试” - 如果您在本地开发代码，然后在虚拟机或另一台服务器上测试它，远程调试是您希望立即启用的功能。

传统上，您将使用这些值修改您的Apache VHost或.htaccess文件：

```
php_value xdebug.remote_host 192.168.?.?
php_value xdebug.remote_port 9000
```

“远程主机”和“远程端口”将对应于您的本地计算机和您配置IDE侦听的端口。然后只需将IDE设置为“侦听连接”模式，并加载URL：

```
http://your-website.example.com/index.php?XDEBUG_SESSION_START=1 
```

当脚本执行时，您的IDE将拦截当前状态，允许您设置断点并探查内存中的值。

图形调试器非常容易逐步执行代码，检查变量并对现有运行时执行代码评估。许多IDE都内置或基于插件支持与Xdebug的图形调试。MacGDBp是一个免费的、开源的、独立的Xdebug GUI，适用于macOS。

# 依赖管理

有大量的 PHP 库、框架和组件可供选择。你的项目可能会使用其中的几个 —— 这些是项目的依赖项。直到最近，PHP 没有一个很好的方法来管理这些项目依赖项。即使你手动管理它们，你仍然需要担心自动加载器。现在这不再是问题了。

目前有两个主要的 PHP 包管理系统 - [Composer](/#composer_and_packagist) 和 [PEAR](/#pear)。Composer 目前是 PHP 中最流行的包管理器，然而在很长一段时间内，PEAR 是主要使用的包管理器。了解 PEAR 的历史是个好主意，因为即使你从未使用过它，你可能仍然会找到与它相关的引用。

## Composer 和 Packagist

Composer 是 PHP 的推荐依赖管理器。在一个`composer.json`文件中列出你项目的依赖项，并通过几个简单的命令，Composer 将自动下载你项目的依赖项并为你设置自动加载。Composer 在 node.js 世界中类似于 NPM，或者在 Ruby 世界中类似于 Bundler。

有大量与 Composer 兼容并准备好在你的项目中使用的 PHP 库。这些“包”在[Packagist](https://packagist.org/)上列出，这是 Composer 兼容的 PHP 库的官方存储库。

### 如何安装 Composer

最安全的下载 Composer 的方法是[按照官方说明](https://getcomposer.org/download/)进行。这将验证安装程序未被损坏或篡改。安装程序会在你的*当前工作目录*中安装一个`composer.phar`二进制文件。

我们建议*全局*安装 Composer（例如，在`/usr/local/bin`中只安装一份）。要做到这一点，请执行以下命令：

```
mv composer.phar /usr/local/bin/composer
```

**注意：** 如果由于权限问题导致上述步骤失败，请在命令前加上`sudo`前缀。

要运行本地安装的 Composer，你需要使用 `php composer.phar`，全局安装则简单地使用 `composer`。

#### 在 Windows 上安装

对于 Windows 用户，最简单的方法是使用[ComposerSetup](https://getcomposer.org/Composer-Setup.exe)安装程序，它执行全局安装并设置了你的`$PATH`，这样你可以在命令行的任何目录中直接调用`composer`。

### 如何定义和安装依赖项

Composer 在一个名为`composer.json`的文件中跟踪你项目的依赖项。你可以手动管理它，或者使用 Composer 本身。`composer require`命令添加一个项目依赖项，如果没有`composer.json`文件，将会创建一个。以下是将[Twig](https://twig.symfony.com/)添加为你项目依赖项的示例。

```
composer require twig/twig:^2.0
```

或者，`composer init`命令将引导你创建一个完整的`composer.json`文件用于你的项目。无论哪种方式，一旦创建了你的`composer.json`文件，你可以告诉 Composer 下载并安装你的依赖项到`vendor/`目录中。这也适用于你已经下载并提供`composer.json`文件的项目：

```
composer install
```

接下来，将此行添加到你的应用程序的主要PHP文件中；这将告诉PHP使用Composer的自动加载器加载你的项目依赖项。

```
<?php
require 'vendor/autoload.php';
```

现在你可以使用你的项目依赖项，并且它们将按需自动加载。

### 更新你的依赖项

Composer会创建一个名为`composer.lock`的文件，其中存储了首次运行`composer install`时下载的每个包的确切版本。如果与他人共享你的项目，请确保包含`composer.lock`文件，这样当他们运行`composer install`时，他们将获得与你相同的版本。要更新你的依赖项，请运行`composer update`。在部署时不要使用`composer update`，只使用`composer install`，否则可能导致生产环境中出现不同的包版本。

这在灵活定义版本需求时非常有用。例如，版本要求为`~1.8`意味着“任何比`1.8.0`更新，但小于`2.0.x-dev`”。你还可以使用`*`通配符，如`1.8.*`。现在，Composer的`composer update`命令将升级所有符合你定义限制的最新版本依赖项。

### 更新通知

要接收有关新版本发布的通知，你可以注册[libraries.io](https://libraries.io/)，这是一个可以监控依赖项并向你发送更新警报的网络服务。

### 检查你的依赖项是否存在安全问题

[本地PHP安全检查器](https://github.com/fabpot/local-php-security-checker)是一个命令行工具，它将检查你的`composer.lock`文件，并告诉你是否需要更新任何依赖项。

### 使用Composer处理全局依赖项

Composer也可以处理全局依赖项及其二进制文件。使用起来很简单，你只需在命令前面加上`global`前缀。例如，如果你想安装PHPUnit并全局可用，可以运行以下命令：

```
composer global require phpunit/phpunit
```

这将创建一个`~/.composer`文件夹，其中存放你的全局依赖项。为了使安装的包的二进制文件在任何地方都可用，你需要将`~/.composer/vendor/bin`文件夹添加到你的`$PATH`变量中。

## PEAR

一些PHP开发者喜欢的老牌包管理器是[PEAR](https://pear.php.net/)。它与Composer类似，但有一些显著的区别。

PEAR要求每个包具有特定的结构，这意味着包的作者必须准备好使其能够与PEAR一起使用。使用未经过PEAR准备的项目是不可能的。

PEAR会全局安装包，这意味着安装一次后，它们就对该服务器上的所有项目都可用。如果两个项目之间存在版本冲突，则可能会出现问题，但如果许多项目依赖于相同版本的同一包，则这可能是一个好处。

### 如何安装PEAR

你可以通过下载`.phar`安装程序并执行它来安装PEAR。PEAR文档详细介绍了每个操作系统的[安装说明](https://pear.php.net/manual/installation.getting.php)。

如果你在使用 Linux，你也可以查看你的发行版包管理器。例如，Debian 和 Ubuntu 提供了一个 apt `php-pear` 包。

### 如何安装一个包

如果软件包在 [PEAR packages list](https://pear.php.net/packages.php) 上列出，你可以通过指定官方名称来安装它：

```
pear install foo
```

如果软件包托管在另一个渠道上，你需要首先`探索`这个渠道，并在安装时指定它。请查看 [Using channel docs](https://pear.php.net/manual/guide.users.commandline.channels.php) 获取更多关于这个主题的信息。

### 使用 Composer 处理 PEAR 依赖

如果你已经在使用 [Composer](/#composer_and_packagist)，并且想要安装一些 PEAR 代码，你可以使用 Composer 来处理你的 PEAR 依赖。Composer 版本 2 不再直接支持 PEAR 仓库，因此你必须手动添加一个仓库来安装 PEAR 包：

```
{  "repositories":  [  {  "type":  "package",  "package":  {  "name":  "pear2/pear2-http-request",  "version":  "2.5.1",  "dist":  {  "url":  "https://github.com/pear2/HTTP_Request/archive/refs/heads/master.zip",  "type":  "zip"  }  }  }  ],  "require":  {  "pear2/pear2-http-request":  "*"  },  "autoload":  {  "psr-4":  {"PEAR2\\HTTP\\":  "vendor/pear2/pear2-http-request/src/HTTP/"}  }  }
```

第一部分 `"repositories"` 将用于让 Composer 知道它应该“初始化”（或在 PEAR 术语中“发现”）pear 仓库。然后 `require` 部分将像这样前缀化包名：

> pear-channel/package

“pear” 前缀是硬编码的，以避免任何冲突，因为一个 pear 渠道可能与另一个包的供应商名称相同，例如，渠道的简短名称（或完整 URL）可以用来引用包所在的渠道。

安装此代码后，它将在您的 vendor 目录中可用，并通过 Composer 自动加载器自动可用。

> vendor/pear2/pear2-http-request/pear2/HTTP/Request.php

要使用这个 PEAR 包，只需像这样引用它：

```
<?php
require __DIR__ . '/vendor/autoload.php';

use PEAR2\HTTP\Request;

$request = new Request();
```

## 基础知识

PHP 是一门广泛的语言，允许所有级别的程序员快速且高效地生成代码。然而，当我们在语言中进步时，我们经常忘记我们最初学到的基础知识（或忽视），而更偏爱于快捷方式和/或坏习惯。为了帮助解决这个常见问题，本节旨在提醒程序员 PHP 中的基本编码实践。

## 日期和时间

PHP 提供了一个名为 DateTime 的类来帮助你在读取、写入、比较或计算日期和时间时使用。除了 DateTime 外，PHP 还有许多与日期和时间相关的函数，但它为大多数常见用法提供了良好的面向对象接口。DateTime 可以处理时区，但这超出了本简短介绍的范围。

要开始使用 DateTime，可以使用 `createFromFormat()` 工厂方法将原始日期和时间字符串转换为对象，或者使用 `new DateTime` 获取当前日期和时间。使用 `format()` 方法将 DateTime 转换回字符串以进行输出。

```
<?php
$raw = '22\. 11\. 1968';
$start = DateTime::createFromFormat('d. m. Y', $raw);

echo 'Start date: ' . $start->format('Y-m-d') . PHP_EOL;
```

使用 DateInterval 类可以进行 DateTime 计算。DateTime 类具有像 `add()` 和 `sub()` 这样的方法，它们以 DateInterval 作为参数。不要编写期望每天有相同秒数的代码。夏令时和时区更改会破坏这一假设。请改用日期间隔。要计算日期差，请使用 `diff()` 方法。它将返回新的 DateInterval，非常容易显示。

```
<?php
// create a copy of $start and add one month and 6 days
$end = clone $start;
$end->add(new DateInterval('P1M6D'));

$diff = $end->diff($start);
echo 'Difference: ' . $diff->format('%m month, %d days (total: %a days)') . PHP_EOL;
// Difference: 1 month, 6 days (total: 37 days)
```

您可以在 DateTime 对象上使用标准比较：

```
<?php
if ($start < $end) {
    echo "Start is before the end!" . PHP_EOL;}
```

最后一个例子是演示 DatePeriod 类。它用于迭代重复事件。它可以接受两个 DateTime 对象，即开始和结束时间，以及它将返回其中所有事件的间隔。

```
<?php
// output all thursdays between $start and $end
$periodInterval = DateInterval::createFromDateString('first thursday');
$periodIterator = new DatePeriod($start, $periodInterval, $end, DatePeriod::EXCLUDE_START_DATE);
foreach ($periodIterator as $date) {
    // output each date in the period
    echo $date->format('Y-m-d') . ' ';
}
```

一个流行的 PHP API 扩展是 [Carbon](https://carbon.nesbot.com/)。它继承了 DateTime 类的所有功能，因此只需进行最少的代码修改，但额外功能包括本地化支持，更多添加、减去和格式化 DateTime 对象的方法，以及通过模拟您选择的日期和时间来测试代码的能力。

## 设计模式

在构建应用程序时，使用代码中的常见模式和项目整体结构的常见模式是有帮助的。使用常见模式有助于更轻松地管理代码，并让其他开发人员快速理解所有内容是如何组合在一起的。

如果您使用框架，那么大多数高层代码和项目结构将基于该框架，因此大部分模式决策已为您决定。但是，您仍然需要选择在框架之上构建的代码中遵循的最佳模式。另一方面，如果您没有使用框架来构建应用程序，则必须找到最适合正在构建的应用程序类型和大小的模式。

您可以在以下地址了解更多关于 PHP 设计模式并查看工作示例：

## 使用 UTF-8

*这部分内容最初由 [Alex Cabal](https://alexcabal.com/) 在 [PHP 最佳实践](https://phpbestpractices.org/#utf-8) 上编写，并被用作我们关于 UTF-8 建议的基础*。

### 没有一句话可以概括。要小心、详细和一致。

目前 PHP 并不在低层支持 Unicode。有方法确保 UTF-8 字符串被正确处理，但这并不容易，并且需要深入研究几乎所有层面的 Web 应用程序，从 HTML 到 SQL 到 PHP。我们将简要概述一下实际情况。

### PHP 层面的 UTF-8

基本的字符串操作，如连接两个字符串和将字符串分配给变量，对于 UTF-8 不需要任何特殊处理。但是，大多数字符串函数，如 `strpos()` 和 `strlen()`，需要特别注意。这些函数通常有一个 `mb_*` 的对应函数：例如，`mb_strpos()` 和 `mb_strlen()`。这些 `mb_*` 字符串函数通过 [多字节字符串扩展](https://www.php.net/book.mbstring) 提供给您，专门用于操作 Unicode 字符串。

操作Unicode字符串时必须使用`mb_*`函数。例如，如果在UTF-8字符串上使用`substr()`，结果很可能包含一些乱码的半个字符。应该使用多字节对应的函数`mb_substr()`。

困难的部分是始终记住在任何时候都要使用`mb_*`函数。即使只是偶尔忘记一次，你的Unicode字符串在进一步处理时也有可能被弄乱。

并非所有的字符串函数都有对应的`mb_*`函数。如果没有你想要的函数，那可能就没有办法了。

你应该在每个你编写的PHP脚本的顶部（或全局包含脚本的顶部）使用`mb_internal_encoding()`函数，并在其后使用`mb_http_output()`函数，如果你的脚本输出到浏览器。在每个脚本中显式定义字符串的编码将会在未来避免许多麻烦。

此外，许多PHP函数在操作字符串时都有一个可选参数，可以指定字符编码。当有这个选项时，你应该始终明确指定UTF-8。例如，`htmlentities()`有一个字符编码选项，如果处理这样的字符串，应始终指定UTF-8。请注意，从PHP 5.4.0开始，`htmlentities()`和`htmlspecialchars()`的默认编码是UTF-8。

最后，如果你正在构建一个分布式应用程序，并且不能确定`mbstring`扩展是否已启用，请考虑使用[Symfony/polyfill-mbstring](https://packagist.org/packages/symfony/polyfill-mbstring) Composer包。如果可用，它将使用`mbstring`，否则将回退到非UTF-8函数。

### 数据库级别的UTF-8

如果你的PHP脚本访问MySQL，即使你遵循了以上所有预防措施，你的字符串也有可能以非UTF-8字符串存储在数据库中。

为确保PHP到MySQL的字符串以UTF-8方式传递，请确保你的数据库和表都设置为`utf8mb4`字符集和校对规则，并在PDO连接字符串中使用`utf8mb4`字符集。见下面的示例代码。这非常重要。

注意，必须使用`utf8mb4`字符集以完全支持UTF-8，而不是`utf8`字符集！详见进一步阅读原因。

### 浏览器级别的UTF-8

使用`mb_http_output()`函数确保你的PHP脚本向浏览器输出UTF-8字符串。

然后，浏览器需要从HTTP响应中得知该页面应被视为UTF-8。今天，通常在HTTP响应头中设置字符集，如下所示：

```
<?php
header('Content-Type: text/html; charset=UTF-8')
```

以前的方法是在页面的`<head>`标签中包含[charset `<meta>`标签](http://htmlpurifier.org/docs/enduser-utf8.html)。

```
<?php
// Tell PHP that we're using UTF-8 strings until the end of the script
mb_internal_encoding('UTF-8');
$utf_set = ini_set('default_charset', 'utf-8');
if (!$utf_set) {
    throw new Exception('could not set default_charset to utf-8, please ensure it\'s set on your system!');
}

// Tell PHP that we'll be outputting UTF-8 to the browser
mb_http_output('UTF-8');

// Our UTF-8 test string
$string = 'Êl síla erin lû e-govaned vîn.';

// Transform the string in some way with a multibyte function
// Note how we cut the string at a non-Ascii character for demonstration purposes
$string = mb_substr($string, 0, 15);

// Connect to a database to store the transformed string
// See the PDO example in this document for more information
// Note the `charset=utf8mb4` in the Data Source Name (DSN)
$link = new PDO(
    'mysql:host=your-hostname;dbname=your-db;charset=utf8mb4',
    'your-username',
    'your-password',
    array(
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_PERSISTENT => false
    )
);

// Store our transformed string as UTF-8 in our database
// Your DB and tables are in the utf8mb4 character set and collation, right?
$handle = $link->prepare('insert into ElvishSentences (Id, Body, Priority) values (default, :body, :priority)');
$handle->bindParam(':body', $string, PDO::PARAM_STR);
$priority = 45;
$handle->bindParam(':priority', $priority, PDO::PARAM_INT); // explicitly tell pdo to expect an int
$handle->execute();

// Retrieve the string we just stored to prove it was stored correctly
$handle = $link->prepare('select * from ElvishSentences where Id = :id');
$id = 7;
$handle->bindParam(':id', $id, PDO::PARAM_INT);
$handle->execute();

// Store the result into an object that we'll output later in our HTML
// This object won't kill your memory because it fetches the data Just-In-Time to
$result = $handle->fetchAll(\PDO::FETCH_OBJ);

// An example wrapper to allow you to escape data to html
function escape_to_html($dirty){
    echo htmlspecialchars($dirty, ENT_QUOTES, 'UTF-8');
}

header('Content-Type: text/html; charset=UTF-8'); // Unnecessary if your default_charset is set to utf-8 already
?><!doctype html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>UTF-8 test page</title>
    </head>
    <body>
        <?php
        foreach($result as $row){
            escape_to_html($row->Body);  // This should correctly output our transformed UTF-8 string to the browser
        }
        ?>
    </body>
</html>
```

### 进一步阅读

## 国际化（i18n）和本地化（l10n）

*新手免责声明：i18n 和 l10n 是数字字母的缩写，一种缩写方式 - 在我们的情况下，国际化变成了 i18n，本地化则变成了 l10n。*

首先，我们需要定义这两个相似的概念和其他相关的事物：

+   **国际化** 是指你组织代码以便于在不同语言或地区进行适应，而无需进行重构。通常情况下，这个动作只需要做一次 - 最好是在项目开始时，否则你可能需要对源代码进行一些重大的更改！

+   **本地化** 发生在你通过翻译内容适应界面（主要是），基于之前进行的 i18n 工作。通常情况下，在新语言或地区需要支持时进行更新，并且当添加新的界面元素时也需要更新，因为它们需要在所有支持的语言中可用。

+   **复数化** 定义了在不同语言之间互操作包含数字和计数器的字符串时所需的规则。例如，在英语中，当你只有一个项目时，它是单数的，而任何不同于此的都被称为复数；在这种语言中，复数通过在一些词后面添加 S 来表示，有时还会改变其部分内容。在其他语言中，如俄语或塞尔维亚语，除了单数外还有两个复数形式 - 甚至可能有四个、五个或六个形式的语言，如斯洛文尼亚语、爱尔兰语或阿拉伯语。

## 实施常见的方式

最简单的国际化 PHP 软件的方法是使用数组文件，并在模板中使用这些字符串，例如 `<h1><?=$TRANS['title_about_page']?></h1>`。然而，这种方法在严肃的项目中并不推荐，因为它在维护过程中可能会遇到一些问题 - 一些问题可能在最初出现，例如复数化。所以，请不要在你的项目中包含超过几页的情况下尝试这样做。

最经典的 i18n 和 l10n 的方式，经常被作为参考的是一种名为 [Gettext 的 Unix 工具](https://en.wikipedia.org/wiki/Gettext)。它可以追溯到 1995 年，至今仍然是一个完整的用于翻译软件的实现。它足够简单易用，同时又具备强大的支持工具。在这里我们将会谈论 Gettext。此外，为了帮助你在命令行中不至于混乱，我们将会介绍一个优秀的 GUI 应用程序，用于轻松更新你的 l10n 源。

有一些常见的库用于支持 Gettext 和其他 i18n 的实现。其中一些可能更容易安装或支持额外的功能或 i18n 文件格式。在这份文件中，我们侧重于 PHP 核心提供的工具，但在这里我们列出其他工具以供参考：

+   [aura/intl](https://github.com/auraphp/Aura.Intl)：提供国际化（I18N）工具，特别是面向包的每地区消息翻译。它使用数组格式的消息。不提供消息提取器，但通过`intl`扩展提供高级消息格式化（包括复数消息）。

+   [php-gettext/Gettext](https://github.com/php-gettext/Gettext)：具有面向对象接口的Gettext支持；包括改进的辅助函数，强大的提取器用于多种文件格式（其中一些格式不被`gettext`命令原生支持），还可以导出到除了`.mo/.po`文件之外的其他格式。如果您需要将翻译文件集成到系统的其他部分（如JavaScript接口），这将非常有用。

+   [symfony/translation](https://symfony.com/zh-cn/components/translation)：支持许多不同的格式，但推荐使用详细的XLIFF格式。不包括助手函数或内置提取器，但通过`strtr()`内部支持占位符。

+   [laminas/laminas-i18n](https://docs.laminas.dev/laminas-i18n/)：支持数组和INI文件，或Gettext格式。实现了缓存层，避免每次读取文件系统。还包括视图助手以及与地区相关的输入过滤器和验证器。然而，它没有消息提取器。

其他框架也包括i18n模块，但这些模块不可在其代码库之外使用：

+   [Laravel](https://laravel.com/docs/master/zh-cn/localization)：支持基本的数组文件，没有自动提取器，但包括用于模板文件的`@lang`助手。

+   [Yii](https://www.yiiframework.com/doc/guide/2.0/zh-cn/tutorial-i18n)：支持数组、Gettext和基于数据库的翻译，并包含消息提取器。它由[`Intl`](https://www.php.net/manual/intl.installation.php)扩展支持，自PHP 5.3起可用，并基于[ICU项目](https://icu.unicode.org/)；这使得Yii能够运行强大的替换，如数字的拼写、日期、时间、间隔、货币和序数格式化。

如果您决定使用不提供提取器的库之一，您可能希望使用Gettext格式，这样您可以像本章剩余部分所述的那样使用原始的Gettext工具链（包括Poedit）。

## Gettext

### 安装

您可能需要通过软件包管理器（如`apt-get`或`yum`）安装Gettext及其相关的PHP库。安装完成后，通过将`extension=gettext.so`（Linux/Unix）或`extension=php_gettext.dll`（Windows）添加到您的`php.ini`中来启用它。

我们还将使用[Poedit](https://poedit.net)来创建翻译文件。您可能会在系统的软件包管理器中找到它；它适用于Unix、macOS和Windows，并且也可以在它们的网站上[免费下载](https://poedit.net/download)。

### 结构

#### 文件类型

在处理 gettext 时，通常会涉及三个文件。主要文件是 PO（可移植对象）和 MO（机器对象）文件，前者是可读的“已翻译对象”的列表，后者则是 gettext 在进行本地化时要解释的对应二进制文件。还有一个 POT（模板）文件，它简单地包含来自源文件的所有现有键，并可用作生成和更新所有 PO 文件的指南。这些模板文件并非强制要求：根据进行本地化的工具，只用 PO/MO 文件也可以完全正常运行。每种语言和地区总会有一对 PO/MO 文件，但每个域只有一个 POT 文件。

### 域

在大型项目中，有时需要根据上下文将相同单词传达不同含义的情况下，您可能需要将其分开为不同的 *域* 进行翻译。它们基本上是命名的 POT/PO/MO 文件组，其中文件名是所述 *翻译域*。小型和中型项目通常出于简单起见只使用一个域；它的名称是任意的，但我们将在代码示例中使用“main”。

#### 区域设置代码

区域设置简单地是标识语言版本的代码。它遵循 [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) 和 [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 规范：两个小写字母表示语言，可选地后跟下划线和两个大写字母表示国家或地区代码。对于 [罕见语言](https://www.gnu.org/software/gettext/manual/gettext.html#Rare-Language-Codes)，使用三个字母。

对于某些讲者来说，国家部分可能看起来是多余的。事实上，一些语言在不同国家有不同的方言，例如奥地利德语（`de_AT`）或巴西葡萄牙语（`pt_BR`）。第二部分用于区分这些方言 - 如果不存在，则被视为语言的“通用”或“混合”版本。

### 目录结构

要使用 Gettext，我们需要遵循特定的文件夹结构。首先，您需要在源代码库中选择一个任意的 l10n 文件根目录。在其中，您将为每个需要的区域设置一个文件夹，并有一个固定的 `LC_MESSAGES` 文件夹，其中包含所有您的 PO/MO 对。例如：

```
<project root>  ├─ src/
 ├─ templates/
 └─ locales/
    ├─ forum.pot
    ├─ site.pot
    ├─ de/
    │  └─ LC_MESSAGES/
    │     ├─ forum.mo
    │     ├─ forum.po
    │     ├─ site.mo
    │     └─ site.po
    ├─ es_ES/
    │  └─ LC_MESSAGES/
    │     └─ ...
    ├─ fr/
    │  └─ ...
    ├─ pt_BR/
    │  └─ ...
    └─ pt_PT/
       └─ ...
```

### 复数形式

正如我们在介绍中所说，不同的语言可能有不同的复数规则。然而，gettext 再次为我们解决了这个麻烦。在创建新的 `.po` 文件时，你需要为该语言声明[复数规则](https://docs.translatehouse.org/projects/localization-guide/en/latest/l10n/pluralforms.html)，而那些与复数相关的翻译片段将根据这些规则的每一种形式而不同。在代码中调用 Gettext 时，你需要指定与句子相关的数字，它会计算出应使用的正确形式 —— 即使需要进行字符串替换也可以。

复数规则包括可用的复数数量以及与 `n` 相关的布尔测试，该测试将定义给定数字落入哪个规则（从 0 开始计数）。例如：

+   日语：`nplurals=1; plural=0` - 只有一条规则

+   英语：`nplurals=2; plural=(n != 1);` - 两条规则，第一条适用于 N 等于一时，第二条适用于其他情况

+   巴西葡萄牙语：`nplurals=2; plural=(n > 1);` - 两条规则，第二条适用于 N 大于一时，第一条适用于其他情况

现在你已经理解了复数规则的基础原理 —— 如果没有，请查看[「LingoHub 教程」](https://lingohub.com/blog/2013/07/php-internationalization-with-gettext-tutorial/#Plurals)上的更深入解释 —— 你可能想要从[列表](https://docs.translatehouse.org/projects/localization-guide/en/latest/l10n/pluralforms.html)中复制所需的复数形式，而不是手动编写它们。

当使用 Gettext 在带有计数器的句子上进行本地化时，你还需要提供相关的数字。Gettext 将计算应生效的规则并使用正确的本地化版本。你需要在 `.po` 文件中为每个定义的复数规则包含不同的句子。

### 示例实现

经过所有这些理论，让我们来实际操作一下。这是一个 `.po` 文件的摘录 —— 不要关注其格式，而是整体内容；稍后你会学会如何轻松编辑它：

```
msgid ""
msgstr ""
"Language: pt_BR\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Plural-Forms: nplurals=2; plural=(n > 1);\n"

msgid "We are now translating some strings"
msgstr "Nós estamos traduzindo algumas strings agora"

msgid "Hello %1$s! Your last visit was on %2$s"
msgstr "Olá %1$s! Sua última visita foi em %2$s"

msgid "Only one unread message"
msgid_plural "%d unread messages"
msgstr[0] "Só uma mensagem não lida"
msgstr[1] "%d mensagens não lidas"
```

第一部分的工作类似于标题，特别是具有空的 `msgid` 和 `msgstr`。它描述了文件编码、复数形式以及其他不那么重要的事情。第二部分将一个简单的字符串从英语翻译成巴西葡萄牙语，第三部分也是如此，但利用了来自 [`sprintf`](https://www.php.net/manual/function.sprintf.php) 的字符串替换，以便翻译中可以包含用户名和访问日期。最后一部分展示了复数形式的示例，显示了英语中的单数和复数版本作为 `msgid`，它们分别作为 `msgstr` 0 和 1 的翻译（遵循由复数规则给出的数字）。在那里，也使用了字符串替换，因此数字可以直接出现在句子中，通过 `%d`。复数形式总是有两个 `msgid`（单数和复数），因此建议不要使用复杂的语言作为翻译源。

### 讨论 l10n 键

正如您可能已经注意到的，我们正在使用实际的英文句子作为源ID。这个`msgid`在所有您的`.po`文件中都是相同的，这意味着其他语言将具有相同的格式和相同的`msgid`字段，但翻译为不同的`msgstr`行。

谈论翻译键时，这里有两个主要的“学派”：

1.  *`msgid`作为一个真实的句子*。其主要优势包括：

    +   如果软件的某些部分在任何给定语言中未翻译，显示的关键内容仍将保持某种意义。例如：如果您从英语翻译到西班牙语但需要帮助翻译到法语，则可能会发布缺少法语句子的新页面，并且网站的某些部分将显示为英语；

    +   对于翻译者来说，理解发生的事情并根据`msgid`进行适当的翻译要容易得多；

    +   它为一种语言（源语言）提供了“免费”的本地化；

    +   唯一的缺点是：如果您需要更改实际文本，则需要在多个语言文件中替换相同的`msgid`。

1.  *`msgid`作为一个唯一的、结构化的关键字*。它会以结构化的方式描述应用程序中的句子角色，包括模板或字符串所在的部分，而不是其内容。

    +   这是一个很好的组织代码的方式，将文本内容与模板逻辑分离。

    +   然而，这可能给翻译者带来问题，因为可能会错过上下文。源语言文件将作为其他翻译的基础。例如：开发人员理想情况下会有一个`en.po`文件，翻译者会阅读该文件以理解在`fr.po`中写什么。

    +   缺少翻译将在屏幕上显示无意义的键（在未翻译的法语页面上显示`top_menu.welcome`而不是`Hello there, User!`）。这对于强制在发布前完成翻译非常有利 - 然而，对界面的翻译问题可能会非常糟糕。但是，一些库包括一个将特定语言指定为“回退”的选项，具有与另一种方法类似的行为。

[Gettext 手册](https://www.gnu.org/software/gettext/manual/gettext.html) 更倾向于第一种方法，因为一般情况下，这对于翻译者和用户在遇到问题时更容易。这也是我们在这里的工作方式。然而，[Symfony 文档](https://symfony.com/doc/current/translation.html#using-real-or-keyword-messages) 更倾向于基于关键字的翻译，以允许独立更改所有翻译而不影响模板。

### 日常使用

在典型的应用程序中，您将在页面上写入静态文本时使用一些Gettext函数。这些句子将出现在`.po`文件中，被翻译，编译成`.mo`文件，然后在渲染实际界面时由Gettext使用。鉴于此，让我们通过一个逐步示例来将我们迄今讨论的内容联系起来：

#### 1\. 一个包含一些不同Gettext调用的样本模板文件

```
<?php include 'i18n_setup.php' ?>
<div id="header">
    <h1><?=sprintf(gettext('Welcome, %s!'), $name)?></h1>
    <!-- code indented this way only for legibility -->
    <?php if ($unread): ?>
        <h2><?=sprintf(
            ngettext('Only one unread message',
                     '%d unread messages',
                     $unread),
            $unread)?>
        </h2>
    <?php endif ?>
</div>

<h1><?=gettext('Introduction')?></h1>
<p><?=gettext('We\'re now translating some strings')?></p>
```

+   [`gettext()`](https://www.php.net/manual/function.gettext.php) 简单地将 `msgid` 翻译为给定语言的对应 `msgstr`。还有缩写函数 `_()` 也是同样的工作方式；

+   [`ngettext()`](https://www.php.net/manual/function.ngettext.php) 也是类似的，但包含了复数规则；

+   还有 [`dgettext()`](https://www.php.net/manual/function.dgettext.php) 和 [`dngettext()`](https://www.php.net/manual/function.dngettext.php)，允许您为单个调用覆盖域配置。关于域配置的更多信息将在下一个示例中讨论；

#### 2\. 一个示例设置文件（如上所述的 `i18n_setup.php`），选择正确的区域设置并配置 Gettext；

```
<?php
/**
 * Verifies if the given $locale is supported in the project
 * @param string $locale
 * @return bool
 */
function valid($locale) {
   return in_array($locale, ['en_US', 'en', 'pt_BR', 'pt', 'es_ES', 'es']);
}

//setting the source/default locale, for informational purposes
$lang = 'en_US';

if (isset($_GET['lang']) && valid($_GET['lang'])) {
    // the locale can be changed through the query-string
    $lang = $_GET['lang'];    //you should sanitize this!
    setcookie('lang', $lang); //it's stored in a cookie so it can be reused
} elseif (isset($_COOKIE['lang']) && valid($_COOKIE['lang'])) {
    // if the cookie is present instead, let's just keep it
    $lang = $_COOKIE['lang']; //you should sanitize this!
} elseif (isset($_SERVER['HTTP_ACCEPT_LANGUAGE'])) {
    // default: look for the languages the browser says the user accepts
    $langs = explode(',', $_SERVER['HTTP_ACCEPT_LANGUAGE']);
    array_walk($langs, function (&$lang) { $lang = strtr(strtok($lang, ';'), ['-' => '_']); });
    foreach ($langs as $browser_lang) {
        if (valid($browser_lang)) {
            $lang = $browser_lang;
            break;
        }
    }
}

// here we define the global system locale given the found language
putenv("LANG=$lang");

// this might be useful for date functions (LC_TIME) or money formatting (LC_MONETARY), for instance
setlocale(LC_ALL, $lang);

// this will make Gettext look for ../locales/<lang>/LC_MESSAGES/main.mo
bindtextdomain('main', '../locales');

// indicates in what encoding the file should be read
bind_textdomain_codeset('main', 'UTF-8');

// if your application has additional domains, as cited before, you should bind them here as well
bindtextdomain('forum', '../locales');
bind_textdomain_codeset('forum', 'UTF-8');

// here we indicate the default domain the gettext() calls will respond to
textdomain('main');

// this would look for the string in forum.mo instead of main.mo
// echo dgettext('forum', 'Welcome back!');
?>
```

#### 3\. 为第一次运行准备翻译；

Gettext 相对于自定义框架的国际化包的一大优势是其广泛且强大的文件格式。“哦，这个相当难以理解和手动编辑，用简单的数组会更容易！” 不要误会，像 [Poedit](https://poedit.net) 这样的应用程序正是来帮助您的 - *很多*。您可以从 [他们的网站](https://poedit.net/download) 下载该程序，它是免费的，适用于所有平台。它是一个非常容易上手的工具，同时也是一个功能强大的工具 - 充分利用了 Gettext 提供的所有功能。本指南基于 PoEdit 1.8 版本；

在第一次运行时，您应该从菜单中选择“文件 > 新建…”。接下来会立即询问您的语言：在这里，您可以选择/筛选您想要翻译的语言，或者使用我们之前提到的格式，例如 `en_US` 或 `pt_BR`；

现在，保存文件 - 使用我们之前提到的目录结构。然后，您应该点击“从源提取”，在这里您将配置提取和翻译任务的各种设置。稍后，您可以通过“目录 > 属性”找到所有这些设置：

+   源路径：这里必须包括项目中调用 `gettext()`（及其衍生函数）的所有文件夹 - 这通常是您的模板/视图文件夹。这是唯一必需的设置；

+   翻译属性：

    +   项目名称和版本，团队及团队的电子邮件地址：这些是放在 .po 文件头部的有用信息；

    +   复数形式：这里是我们之前提到的规则 - 链接中也有示例。大多数情况下，您可以使用默认选项，因为 PoEdit 已经包含了许多语言的复数规则数据库；

    +   字符集：首选 UTF-8；

    +   源代码字符集：在这里设置您的代码库使用的字符集 - 可能也是 UTF-8，对吧？

+   源关键词：底层软件知道多种编程语言中 `gettext()` 和类似函数调用的样子，但您也可以创建自己的翻译函数。这将在“提示”部分后面讨论；

在设置了这些点之后，它将通过您的源文件运行扫描，以查找所有的本地化调用。每次扫描后，PoEdit 将显示一个摘要，显示找到了什么，从源文件中删除了什么。新条目将被空置到翻译表中，然后您将开始键入这些字符串的本地化版本。保存后，将会编译一个 .mo 文件到同一文件夹中，然后，您的项目已国际化完成。

#### 4\. 翻译字符串

正如您之前可能已经注意到的，本地化字符串有两种主要类型：简单的和有复数形式的。前者只有两个框：源和本地化字符串。源字符串不能修改，因为 Gettext/Poedit 没有修改源文件的权限 - 您应该直接更改源文件并重新扫描文件。提示：您可以右键点击一个翻译行，它将提示您源文件和使用该字符串的行数。另一方面，复数形式字符串包括两个框来显示两个源字符串，并且有选项卡可以配置不同的最终形式。

每当您更改源文件并需要更新翻译时，只需点击“刷新”，Poedit 将重新扫描代码，删除不存在的条目，合并已更改的条目并添加新条目。它还可能尝试猜测一些翻译，基于您已经做过的其他猜测和更改的条目。这些猜测和更改的条目将会带有“模糊”标记，表示需要审核，在列表中显示为金黄色。如果您有一个翻译团队，而有人试图写一些他们不确定的东西，这也非常有用：只需标记为“模糊”，稍后会有其他人进行审核。

最后，建议保持“查看 > 首先显示未翻译的条目”选中，这将帮助您*很多*，以免遗漏任何条目。从那个菜单中，您还可以打开 UI 的部分，如果需要的话，可以为翻译人员留下上下文信息。

### 小技巧

#### 可能的缓存问题

如果您在 Apache 上运行 PHP 作为模块（`mod_php`），您可能会遇到`.mo`文件被缓存的问题。第一次读取时会发生这种情况，然后要更新它，可能需要重新启动服务器。在 Nginx 和 PHP5 上，通常只需刷新几次页面即可刷新翻译缓存，而在 PHP7 上则很少需要。

#### 额外的辅助函数

正如许多人所偏爱的，使用`_()`而不是`gettext()`更容易。许多来自框架的自定义 i18n 库也使用类似于`t()`的东西，以缩短翻译后的代码。然而，这是唯一一个支持快捷方式的函数。您可能希望在项目中添加一些其他函数，例如`__()`或`_n()`用于`ngettext()`，或者可能是一个复杂的`_r()`，它将`gettext()`和`sprintf()`调用结合起来。其他库，如[php-gettext 的 Gettext](https://github.com/php-gettext/Gettext)，也提供类似的辅助函数。

在这些情况下，您需要告知Gettext工具如何从这些新函数中提取字符串。不要担心，这很简单。这只是`.po`文件中的一个字段，或者Poedit上的设置屏幕。在编辑器中，该选项位于“Catalog > Properties > Source keywords”内。记住：Gettext已经知道许多语言的默认函数，因此如果列表看起来空无一物，也不必担心。您需要在那里包含这些新函数的规范，遵循[a specific format](https://www.gnu.org/software/gettext/manual/gettext.html#Language-specific-options)。

+   如果您创建类似于`t()`的函数，它只需返回字符串的翻译，您可以将其指定为`t`。Gettext会知道唯一的函数参数是要翻译的字符串；

+   如果函数有多个参数，您可以指定第一个字符串在哪个参数中，以及必要时，复数形式也是如此。例如，如果我们像这样调用我们的函数：`__('one user', '%d users', $number)`，规范将是`__:1,2`，意味着第一个形式是第一个参数，第二个形式是第二个参数。如果您的数字作为第一个参数出现，规范将是`__:2,3`，表示第一个形式是第二个参数，依此类推。

在`.po`文件中包含这些新规则后，新的扫描将像以前一样轻松地引入您的新字符串。

### 参考资料

# 依赖注入

来自[Wikipedia](https://wikipedia.org/wiki/Dependency_injection):

> 依赖注入是一种软件设计模式，允许移除硬编码的依赖关系，并且可以在运行时或编译时进行更改。

这句话使得这个概念听起来比实际情况复杂得多。依赖注入是通过构造函数注入、方法调用或属性设置来提供组件及其依赖关系。就是这么简单。

## 基本概念

我们可以通过一个简单而天真的例子来演示这个概念。

这里有一个`Database`类，它需要一个适配器来与数据库通信。我们在构造函数中实例化适配器并创建了一个硬依赖关系。这使得测试变得困难，并且意味着`Database`类与适配器紧密耦合。

```
<?php
namespace Database;

class Database
{
    protected $adapter;

    public function __construct()
    {
        $this->adapter = new MySqlAdapter;
    }
}

class MysqlAdapter {}
```

这段代码可以重构以使用依赖注入，从而减少依赖关系。在这里，我们通过构造函数注入依赖，并使用[constructor property promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion)，使其作为类中的属性可用：

```
<?php
namespace Database;

class Database
{
    public function __construct(protected MySqlAdapter $adapter)
    {
    }
}

class MysqlAdapter {}
```

现在，我们给`Database`类提供其依赖项，而不是自己创建它。我们甚至可以创建一个方法，接受依赖项作为参数并设置它，或者如果`$adapter`属性是`public`，我们可以直接设置它。

## 复杂问题

如果你曾经阅读过有关依赖注入的内容，那么你可能已经看到过术语*“控制反转”*或*“依赖反转原则”*。这些是依赖注入解决的复杂问题。

### 控制反转

控制反转就像它所说的，“反转控制”一个系统，通过将组织控制与我们的对象完全分开。在依赖注入方面，这意味着通过在系统的其他位置控制和实例化它们来放松我们的依赖关系。

多年来，PHP框架一直在实现控制反转，但问题是，我们正在反转控制的哪一部分，以及反转到哪里？例如，MVC框架通常会提供一个超级对象或基础控制器，其他控制器必须扩展它才能访问其依赖项。这**是**控制反转，但与放松依赖项不同，这种方法只是移动了它们。

依赖注入允许我们通过仅在需要时注入我们需要的依赖项来更优雅地解决这个问题，而不需要任何硬编码的依赖项。

### S.O.L.I.D.

#### 单一职责原则

单一职责原则关于角色和高级架构。它指出：“一个类应该只有一个变化的原因。” 这意味着每个类应该*仅*负责软件提供的功能的一部分。这种方法的最大好处在于它能够提升代码的*可重用性*。通过设计我们的类只做一件事，我们可以在任何其他程序中使用（或重复使用）它，而无需更改它。

#### 开放/封闭原则

开放/封闭原则关于类设计和功能扩展。它指出：“软件实体（类、模块、函数等）应该对扩展开放，但对修改关闭。” 这意味着我们应该设计我们的模块、类和函数，以便在需要新功能时，不修改现有代码，而是编写新代码，供现有代码使用。从实际角度来看，这意味着我们应该编写实现并遵循*接口*的类，然后对这些接口进行类型提示，而不是特定的类。

这种方法的最大好处在于，我们可以非常轻松地通过添加对新功能的支持来扩展我们的代码，而无需修改现有代码，这意味着我们可以减少QA时间，并且大大降低对应用程序的负面影响的风险。我们可以更快地部署新代码，并且更有信心。

#### 里氏替换原则

里氏替换原则关于子类型化和继承。它指出：“子类不应该破坏父类的类型定义。” 或者用罗伯特·C·马丁的话来说，“子类型必须能够替换它们的基类型。”

例如，如果我们有一个定义了`FileInterface`接口并定义了`embed()`方法，以及我们有`Audio`和`Video`类都实现了`FileInterface`接口，那么我们可以期望使用`embed()`方法总是能达到我们的意图。如果我们后来创建了一个实现`FileInterface`接口的`PDF`类或`Gist`类，我们已经了解并理解了`embed()`方法将会做什么。这种方法的最大好处是我们能够构建灵活且易于配置的程序，因为当我们将一个类型（例如`FileInterface`）的一个对象更改为另一个对象时，我们无需在程序中进行任何其他更改。

#### 接口隔离原则

接口隔离原则（ISP）关注的是*业务逻辑对客户端的通信*。它指出“不应该强迫任何客户端依赖它们不使用的方法。” 这意味着，我们不应该有一个单一的庞大接口需要所有符合的类都去实现，而是应该提供一组更小、概念特定的接口，符合的类可以实现其中一个或多个。

例如，`Car`或`Bus`类可能对`steeringWheel()`方法感兴趣，但`Motorcycle`或`Tricycle`类则不是。相反，`Motorcycle`或`Tricycle`类可能对`handlebars()`方法感兴趣，但`Car`或`Bus`类则不是。没有必要让所有这些类型的车辆都实现对`steeringWheel()`和`handlebars()`的支持，因此我们应该拆分源接口。

#### 依赖倒置原则

依赖倒置原则是关于消除离散类之间的硬链接，以便通过传递不同的类来利用新功能。它指出我们应该*“依赖于抽象，不要依赖于具体实现。”* 简而言之，这意味着我们的依赖应该是接口/契约或抽象类，而不是具体的实现。我们可以轻松地重构上面的例子以遵循这一原则。

```
<?php
namespace Database;

class Database
{
    public function __construct(protected AdapterInterface $adapter)
    {
    }
}

interface AdapterInterface {}

class MysqlAdapter implements AdapterInterface {}
```

现在`Database`类依赖于接口而不是具体实现有几个好处。

考虑到我们正在一个团队中工作，并且适配器正在由同事开发。在我们的第一个例子中，我们将不得不等到该同事完成适配器才能为我们的单元测试正确模拟它。现在依赖于接口/契约，我们可以愉快地模拟该接口，知道我们的同事将基于该契约构建适配器。

使用这种方法的另一个更大的好处是，我们的代码现在更具扩展性。如果一年后我们决定要迁移到不同类型的数据库，我们可以编写一个实现原始接口的适配器，并注入它，不需要进行其他的重构，因为我们可以确保适配器遵循接口设置的契约。

## 容器

关于依赖注入容器的第一件事是要理解它们与依赖注入不是同一回事。容器是一个方便的实用程序，帮助我们实现依赖注入，然而，它们经常被误用来实现反模式的服务定位。向类中注入一个DI容器作为服务定位器，可以说比替换的依赖性更加依赖容器。这也会使您的代码更加不透明，最终更难测试。

大多数现代框架都有自己的依赖注入容器，允许您通过配置将依赖项连接在一起。在实践中，这意味着您可以编写与构建在其上的框架一样干净和解耦的应用程序代码。

# 数据库

很多时候，您的PHP代码会使用数据库来持久化信息。您有几个选项来连接和与数据库交互。在PHP 5.1.0之前，推荐的选项是使用诸如[mysqli](https://www.php.net/mysqli)、[pgsql](https://www.php.net/pgsql)、[mssql](https://www.php.net/mssql)等原生驱动程序。

如果您的应用程序只使用*一个*数据库，原生驱动程序是非常好的选择。但是，如果例如您同时使用MySQL和少量MSSQL，或者需要连接Oracle数据库，那么您将无法使用相同的驱动程序。您需要为每个数据库学习全新的API — 这可能有点儿愚蠢。

## MySQL扩展

PHP的[mysql](https://www.php.net/mysqli)扩展非常古老，已经被其他两个扩展取代：

[mysql](https://www.php.net/mysqli)已经很久没有进行开发了，而且**已经[在PHP 7.0中被官方移除了](https://www.php.net/manual/migration70.removed-exts-sapis.php)**。

为了避免深入`php.ini`设置来查看您正在使用哪个模块，一个选择是在您选择的编辑器中搜索`mysql_*`。如果出现任何`mysql_connect()`和`mysql_query()`等函数，则表示正在使用`mysql`。

即使您还没有开始使用PHP 7.x或更高版本，如果不尽快考虑这种升级，当PHP升级到这些版本时会更加困难。最佳选择是在您自己的开发进度表内，将应用程序中的mysql使用替换为[mysqli](https://www.php.net/mysqli)或[PDO](https://www.php.net/pdo)，这样以后就不会被赶时间。

**如果您正从[mysql](https://www.php.net/mysqli)升级到[mysqli](https://www.php.net/mysqli)，请注意那些建议您只需简单地将`mysql_*`替换为`mysqli_*`的懒惰升级指南。这不仅是对问题的过度简化，还忽略了mysqli提供的参数绑定等优势，这些也同样适用于[PDO](https://www.php.net/pdo)。**

## PDO扩展

[PDO](https://www.php.net/pdo)是一个数据库连接抽象库 —— 自PHP 5.1.0起内置于PHP中 —— 提供了一个通用的接口来与许多不同的数据库交互。例如，你可以使用基本相同的代码与MySQL或SQLite进行交互：

```
<?php
// PDO + MySQL
$pdo = new PDO('mysql:host=example.com;dbname=database', 'user', 'password');
$statement = $pdo->query("SELECT some_field FROM some_table");
$row = $statement->fetch(PDO::FETCH_ASSOC);
echo htmlentities($row['some_field']);

// PDO + SQLite
$pdo = new PDO('sqlite:/path/db/foo.sqlite');
$statement = $pdo->query("SELECT some_field FROM some_table");
$row = $statement->fetch(PDO::FETCH_ASSOC);
echo htmlentities($row['some_field']);
```

PDO不会翻译你的SQL查询或模拟缺失的功能；它纯粹是为了使用相同的API连接到多种类型的数据库。

更重要的是，`PDO`允许你安全地将外部输入（例如ID）注入到SQL查询中，而无需担心数据库SQL注入攻击。这是通过PDO语句和绑定参数实现的。

假设一个PHP脚本接收一个数字ID作为查询参数。这个ID应该用于从数据库中获取用户记录。这种方式是`错误的`：

```
<?php
$pdo = new PDO('sqlite:/path/db/users.db');
$pdo->query("SELECT name FROM users WHERE id = " . $_GET['id']); // <-- NO!
```

这是糟糕的代码。你正在将一个原始的查询参数插入到SQL查询中。这将使你很快被黑客攻击，使用一种称为[SQL注入](https://web.archive.org/web/20210413233627/http://wiki.hashphp.org/Validation)的做法。想象一下，如果黑客通过调用类似`http://domain.com/?id=1%3BDELETE+FROM+users`的URL传入一个巧妙的`id`参数，这将设置`$_GET['id']`变量为`1;DELETE FROM users`，这将删除所有用户！相反，你应该使用PDO绑定参数来清理ID输入。

```
<?php
$pdo = new PDO('sqlite:/path/db/users.db');
$stmt = $pdo->prepare('SELECT name FROM users WHERE id = :id');
$id = filter_input(INPUT_GET, 'id', FILTER_SANITIZE_NUMBER_INT); // <-- filter your data first (see [Data Filtering](#data_filtering)), especially important for INSERT, UPDATE, etc.
$stmt->bindParam(':id', $id, PDO::PARAM_INT); // <-- Automatically sanitized for SQL by PDO
$stmt->execute();
```

这是正确的代码。它在PDO语句上使用了绑定参数。这样可以在将外部输入的ID引入数据库之前对其进行转义，从而防止潜在的SQL注入攻击。

对于写操作，例如INSERT或UPDATE，仍然特别关键的是首先对数据进行[过滤](#data_filtering)并对其进行净化以供其他用途（去除HTML标记、JavaScript等）。PDO只会为SQL进行净化，而不是为你的应用程序。

你还应该意识到数据库连接会消耗资源，如果连接没有被显式关闭，则可能会耗尽资源，不过这在其他语言中更为常见。使用PDO，你可以通过销毁对象并确保删除所有对它的引用（即设置为NULL）来隐式关闭连接。如果你不显式地这样做，PHP会在脚本结束时自动关闭连接 - 当然，如果你使用的是持久连接，则除外。

## 与数据库交互

当开发者初学PHP时，他们经常会将他们的数据库交互与他们的呈现逻辑混合在一起，使用看起来像这样的代码：

```
<ul>
<?php
foreach ($db->query('SELECT * FROM table') as $row) {
    echo "<li>".$row['field1']." - ".$row['field1']."</li>";
}
?>
</ul>
```

出于各种原因，这都是一个糟糕的做法，主要是难以调试、难以测试、难以阅读，如果不设置限制，它会输出大量字段。

虽然还有许多其他解决方案可以完成这个任务 - 取决于你是偏向于[面向对象编程（OOP）](/#object-oriented-programming)还是[函数式编程](/#functional-programming) - 但必须要有一定的分离元素。

考虑最基本的步骤：

```
<?php
function getAllFoos($db) {
    return $db->query('SELECT * FROM table');
}

$results = getAllFoos($db);
foreach ($results as $row) {
    echo "<li>".$row['field1']." - ".$row['field1']."</li>"; // BAD!!
}
```

这是一个良好的开始。将这两个项目放在不同的文件中，你就能实现一些清晰的分离。

创建一个类来放置该方法，并将其称为“模型”。创建一个简单的`.php`文件来放置表现逻辑，并将其称为“视图”，这几乎就是[MVC](https://code.tutsplus.com/tutorials/mvc-for-noobs--net-10488)——大多数[框架](/#frameworks)常见的面向对象编程架构。

**foo.php**

```
<?php
$db = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8mb4', 'username', 'password');

// Make your model available
include 'models/FooModel.php';

// Create an instance
$fooModel = new FooModel($db);
// Get the list of Foos
$fooList = $fooModel->getAllFoos();

// Show the view
include 'views/foo-list.php';
```

**models/FooModel.php**

```
<?php
class FooModel
{
    public function __construct(protected PDO $db)
    {
    }

    public function getAllFoos() {
        return $this->db->query('SELECT * FROM table');
    }
}
```

**views/foo-list.php**

```
<?php foreach ($fooList as $row): ?>
    <li><?= $row['field1'] ?> - <?= $row['field1'] ?></li>
<?php endforeach ?>
```

这与大多数现代框架所做的基本相同，尽管有些手动。你可能不需要每次都做完所有这些，但是如果你希望对你的应用进行[unit-test](/#unit-testing)，那么混合太多的表现逻辑和数据库交互可能会成为一个真正的问题。

## 抽象层

许多框架提供自己的抽象层，这些抽象层可能会或可能不会基于[PDO](https://www.php.net/book.pdo)。这些通常通过将你的查询包装在PHP方法中来模拟一个数据库系统的特性，这不仅仅是连接抽象，而是真正的数据库抽象。当然，这会增加一些开销，但如果你正在构建一个需要与MySQL、PostgreSQL和SQLite一起工作的便携式应用程序，为了代码的整洁性，稍微的额外开销是值得的。

一些抽象层是使用[PSR-0](https://www.php-fig.org/psr/psr-0/)或[PSR-4](https://www.php-fig.org/psr/psr-4/)命名空间标准构建的，因此可以安装在任何您喜欢的应用程序中：

# 模板化

模板提供了一种方便的方式来分离控制器和领域逻辑与表现逻辑。模板通常包含应用程序的HTML，但也可以用于其他格式，如XML。模板通常被称为“视图”，这是[model-view-controller](/pages/Design-Patterns.html#model-view-controller)（MVC）软件架构模式的第二组成部分之一。

## 好处

使用模板的主要好处是它们在创建表现逻辑与应用程序其余部分之间的清晰分离。模板的唯一责任是显示格式化内容。它们不负责数据查找、持久性或其他更复杂的任务。这导致更干净、更可读的代码，这在团队环境中特别有帮助，开发人员专注于服务器端代码（控制器、模型），而设计师专注于客户端代码（标记）。

模板还改善了表现代码的组织。模板通常放置在“views”文件夹中，每个模板定义在单个文件中。这种方法鼓励代码重用，其中较大的代码块被分解为较小、可重用的片段，通常称为partials。例如，您的网站页眉和页脚可以各自被定义为模板，然后在每个页面模板之前和之后包含它们。

最后，根据您使用的库不同，模板可以通过自动转义用户生成的内容来提供更多安全性。一些库甚至提供沙箱功能，模板设计者只能访问白名单中的变量和函数。

## 简单的PHP模板

简单的PHP模板只是使用原生PHP代码的模板。由于PHP本身实际上就是一个模板语言，这是一个自然的选择。这意味着您可以在其他代码中组合PHP代码，比如HTML。这对PHP开发者非常有益，因为他们无需学习新的语法，他们了解可用于他们的功能，并且他们的代码编辑器已经内置了PHP语法高亮显示和自动完成。此外，纯PHP模板往往非常快速，因为不需要编译阶段。

每个现代PHP框架都采用某种模板系统，其中大多数默认使用纯PHP。在框架之外，像[Plates](https://platesphp.com/)或[Aura.View](https://github.com/auraphp/Aura.View)这样的库通过提供继承、布局和扩展等现代模板功能，使得使用纯PHP模板更加简便。

### 简单的纯PHP模板示例

使用[Plates](https://platesphp.com/)库。

```
<?php // user_profile.php ?>

<?php $this->insert('header', ['title' => 'User Profile']) ?>

<h1>User Profile</h1>
<p>Hello, <?=$this->escape($name)?></p>

<?php $this->insert('footer') ?>
```

### 纯PHP模板使用继承的示例

使用[Plates](https://platesphp.com/)库。

```
<?php // template.php ?>

<html>
<head>
    <title><?=$title?></title>
</head>
<body>

<main>
    <?=$this->section('content')?>
</main>

</body>
</html>
```

```
<?php // user_profile.php ?>

<?php $this->layout('template', ['title' => 'User Profile']) ?>

<h1>User Profile</h1>
<p>Hello, <?=$this->escape($name)?></p>
```

## 编译模板

尽管PHP已经发展成为一种成熟的面向对象语言，但作为模板语言它并[没有太大改进](http://fabien.potencier.org/templating-engines-in-php.html)。编译模板，如[Twig](https://twig.symfony.com/)、[Brainy](https://github.com/box/brainy)或[Smarty](https://www.smarty.net/)，填补了这一空白，通过提供一种专门用于模板的新语法。从自动转义到继承和简化的控制结构，编译模板设计为更易于编写、更清晰易读和更安全使用。编译模板甚至可以跨不同语言共享，[Mustache](https://mustache.github.io/)就是一个很好的例子。由于这些模板必须编译，因此会有轻微的性能损失，但是如果使用适当的缓存，这种损失非常小。

**虽然Smarty提供自动转义，但默认情况下这个功能是未启用的。*

### 编译模板的简单示例

使用[Twig](https://twig.symfony.com/)库。

```
{% include 'header.html' with {'title': 'User Profile'} %}

<h1>User Profile</h1>
<p>Hello, {{ name }}</p>

{% include 'footer.html' %}
```

### 使用继承的编译模板示例

使用[Twig](https://twig.symfony.com/)库。

```
// template.html

<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>

<main>
    {% block content %}{% endblock %}
</main>

</body>
</html>
```

```
// user_profile.html

{% extends "template.html" %}

{% block title %}User Profile{% endblock %}
{% block content %}
    <h1>User Profile</h1>
    <p>Hello, {{ name }}</p>
{% endblock %}
```

## 进一步阅读

### 文章和教程

### 库

## 错误

在许多“异常密集”的编程语言中，无论何时出现问题，都会抛出异常。这当然是一种可行的做法，但PHP是一种“异常较轻”的编程语言。尽管它确实有异常并且在处理对象时核心部分开始使用它们，但PHP本身大部分情况下会尝试继续处理，不管发生了什么，除非发生致命错误。

例如：

```
$  php -a
php >  echo $foo;
Notice: Undefined variable: foo in php shell code on line 1
```

这只是一个通知错误，PHP 会继续执行。对于那些来自“异常密集型”语言的人来说，这可能会感到困惑，因为例如在 Python 中引用一个丢失的变量会抛出异常：

```
$  python
>>> print foo
Traceback (most recent call last):
 File "<stdin>", line 1, in <module> NameError: name 'foo' is not defined
```

唯一真正的区别在于 Python 会因为任何小问题而感到恐慌，以便开发人员可以非常确保捕捉到任何潜在问题或边缘情况，而 PHP 会继续处理，除非发生极端情况，此时它会抛出错误并报告它。

### 错误严重程度

PHP 有几个错误严重级别。最常见的三种消息类型是错误、通知和警告。它们有不同的严重程度级别；`E_ERROR`、`E_NOTICE` 和 `E_WARNING`。错误是致命的运行时错误，通常由代码中的故障引起，需要修复，因为它们将导致 PHP 停止执行。通知是由代码引起的建议性消息，在脚本执行过程中可能会或可能不会引起问题，执行不会停止。警告是非致命错误，脚本的执行不会停止。

在编译时报告的另一种类型的错误消息是 `E_STRICT` 消息。这些消息用于建议更改您的代码，以帮助确保与即将推出的 PHP 版本的最佳互操作性和向前兼容性。

### 更改 PHP 的错误报告行为

通过使用 PHP 设置和/或 PHP 函数调用，可以更改错误报告。使用内置的 PHP 函数 `error_reporting()` 可以通过传递预定义的错误级别常量来设置脚本执行期间的错误级别，这意味着如果你只想看到错误和警告 - 而不是通知 - 那么你可以配置如下：

```
<?php
error_reporting(E_ERROR | E_WARNING);
```

你还可以控制错误是否显示在屏幕上（适用于开发）或隐藏，并记录（适用于生产）。有关更多信息，请查看[错误报告](/#error_reporting)部分。

### 内联错误抑制

你也可以使用 PHP 的错误控制运算符 `@` 来抑制特定的错误。将该运算符放在表达式的开头，任何由该表达式直接引起的错误都会被静音。

```
<?php
echo @$foo['bar'];
```

如果存在，则输出 `$foo['bar']`，但如果变量 `$foo` 或 `'bar'` 键不存在，则只会返回 null 并且什么也不会打印。没有错误控制运算符，此表达式可能会创建 `PHP Notice: Undefined variable: foo` 或 `PHP Notice: Undefined index: bar` 错误。

这可能看起来是个好主意，但有一些不太理想的折衷方案。PHP 处理带有 `@` 的表达式的性能不如没有 `@` 的表达式高。过早进行优化可能是所有编程争论的根源，但如果性能对你的应用程序/库特别重要，理解错误控制运算符的性能影响是很重要的。

其次，错误控制运算符**完全**吞没了错误。错误不会显示出来，也不会发送到错误日志中。此外，库存/生产PHP系统无法关闭错误控制运算符。虽然您可能认为看到的错误是无害的，但另一种更不利的错误同样会悄无声息。

如果有方法可以避免错误抑制运算符，您应该考虑使用它。例如，我们的上述代码可以重写为：

```
<?php
// Null Coalescing Operator
echo $foo['bar'] ?? '';
```

一个例子是，错误抑制可能是有意义的地方是当`fopen()`无法找到要加载的文件时。您可以在尝试加载之前检查文件是否存在，但如果在检查之后和`fopen()`之前删除了文件（这听起来可能不可能，但确实可能发生），那么`fopen()`将返回false *并*抛出错误。这可能是PHP应该解决的问题，但这是一种情况，错误抑制似乎是唯一有效的解决方案。

之前我们提到过，在标准PHP系统中没有办法关闭错误控制运算符。然而，[Xdebug](https://xdebug.org/docs/basic)有一个`xdebug.scream` ini设置，它将禁用错误控制运算符。您可以通过您的`php.ini`文件设置如下。

```
xdebug.scream = On
```

您还可以使用`ini_set`函数在运行时设置此值。

```
<?php
ini_set('xdebug.scream', '1')
```

在调试代码并怀疑信息错误被抑制时，这是非常有用的。谨慎使用`scream`作为临时调试工具。许多PHP库代码可能无法在禁用错误控制运算符的情况下正常工作。

### ErrorException

PHP完全有能力成为一个“异常重”的编程语言，只需几行代码即可切换。基本上，您可以使用`ErrorException`类将您的“错误”抛出为“异常”，该类扩展了`Exception`类。

这是许多现代框架（如Symfony和Laravel）实施的常见做法。在调试模式（*或开发模式*）下，这些框架将显示一个漂亮而干净的*堆栈跟踪*。

还有一些可用于更好的错误和异常处理和报告的包。例如，[Whoops!](https://filp.github.io/whoops/)，它默认安装在Laravel中，并且可以在任何框架中使用。

在开发中通过将错误抛出为异常，您可以比通常的结果更好地处理它们，如果在开发过程中看到异常，您可以在catch语句中包装它，并提供特定的处理说明。每个捕获的异常都使您的应用程序更加健壮一些。

更多关于如何使用`ErrorException`进行错误处理的信息和详细信息可在[ErrorException Class](https://www.php.net/class.errorexception)找到。

## 异常

异常是大多数流行编程语言的标准组成部分，但它们经常被 PHP 程序员忽视。像 Ruby 这样的语言非常重视异常，所以每当出现问题，比如 HTTP 请求失败、数据库查询出错，甚至是找不到图像资源时，Ruby（或使用的 gem）会将异常抛到屏幕上，这意味着您立即知道出了什么问题。

PHP 本身在这方面相对宽松，调用 `file_get_contents()` 通常只会得到一个 `FALSE` 和一个警告。许多旧的 PHP 框架如 CodeIgniter 只会返回一个 false，并记录一条消息到它们的专有日志中，也许让您使用像 `$this->upload->get_error()` 这样的方法来查看出了什么问题。这里的问题在于您必须寻找错误并查看文档以了解该类的错误方法，而不是使错误变得非常明显。

另一个问题是当类自动向屏幕抛出错误并退出进程时。当您这样做时，您阻止了另一位开发人员能够动态处理该错误。应该抛出异常以使开发人员意识到错误；然后他们可以选择如何处理这个错误。例如：

```
<?php
$email = new Fuel\Email;
$email->subject('My Subject');
$email->body('How the heck are you?');
$email->to('guy@example.com', 'Some Guy');

try
{
    $email->send();
}
catch(Fuel\Email\ValidationFailedException $e)
{
    // The validation failed
}
catch(Fuel\Email\SendingFailedException $e)
{
    // The driver could not send the email
}
finally
{
    // Executed regardless of whether an exception has been thrown, and before normal execution resumes
}
```

### SPL 异常

通用的 `Exception` 类为开发人员提供了非常少的调试上下文；但是，为了解决这个问题，可以通过继承通用的 `Exception` 类来创建专门的 `Exception` 类型：

```
<?php
class ValidationException extends Exception {}
```

这意味着您可以添加多个 catch 块并以不同方式处理不同的异常。这可能导致创建大量自定义异常，其中一些可以通过使用[SPL 扩展](/#standard_php_library)中提供的 SPL 异常来避免。

例如，如果您使用 `__call()` 魔术方法并请求了无效方法，而不是抛出一个模糊的标准异常，或者专门为此创建一个自定义异常，您可以简单地 `throw new BadMethodCallException;`。

## Web 应用安全

对每个 PHP 开发人员来说，学习[Web 应用安全的基础知识](https://paragonie.com/blog/2015/08/gentle-introduction-application-security)非常重要，可以分解为几个广泛的主题：

1.  代码-数据分离。

    +   当数据作为代码执行时，会导致 SQL 注入、跨站点脚本攻击、本地/远程文件包含等问题。

    +   当代码被打印为数据时，会发生信息泄露（源代码泄露或在 C 程序中足够绕过 [ASLR](https://www.techtarget.com/searchsecurity/definition/address-space-layout-randomization-ASLR) 的信息）。

1.  应用程序逻辑。

    +   缺失的身份验证或授权控制。

    +   输入验证。

1.  操作环境。

    +   PHP 版本。

    +   第三方库。

    +   操作系统。

1.  密码学弱点。

有些人准备好并愿意利用您的 Web 应用程序。重要的是您采取必要的预防措施来加固 Web 应用程序的安全性。幸运的是，[开放 Web 应用程序安全项目](https://www.owasp.org/)（OWASP）的专家们已编制了一份详尽的已知安全问题列表以及保护自己的方法。对于注重安全的开发者来说，这是必读之选。Padraic Brady 的 [Survive The Deep End: PHP 安全](https://phpsecurity.readthedocs.io/en/latest/index.html) 也是 PHP Web 应用程序安全指南的好书。

## 密码哈希

最终每个人都会构建一个依赖于用户登录的 PHP 应用程序。用户名和密码被存储在数据库中，并在登录时用于验证用户身份。

对于存储密码，[*哈希*](https://wikipedia.org/wiki/Cryptographic_hash_function) 是非常重要的，存储之前需要正确地进行哈希处理。哈希和加密是[两个非常不同的概念](https://paragonie.com/blog/2015/08/you-wouldnt-base64-a-password-cryptography-decoded)，但经常会被混淆。

哈希是不可逆的单向函数。它生成一个固定长度的字符串，不能被轻易逆转。这意味着你可以比较一个哈希和另一个哈希，以确定它们是否来自同一源字符串，但你无法确定原始字符串。如果密码未经过哈希处理，你的数据库被未经授权的第三方访问，所有用户帐户都会受到威胁。

与哈希不同，加密是可逆的（如果您有密钥的话）。加密在其他领域中很有用，但对于安全存储密码来说则是一个很差的策略。

密码还应该通过在每个密码前添加一个随机字符串进行[*盐化*](https://wikipedia.org/wiki/Salt_(cryptography))，然后再进行哈希处理。这可以防止字典攻击和使用“彩虹表”（一种常见密码的密码哈希的反向列表）。

哈希和盐化非常重要，因为用户经常在多个服务中使用相同的密码，密码质量可能很差。

此外，您应使用[a specialized *password hashing* algorithm](https://paragonie.com/blog/2016/02/how-safely-store-password-in-2016)而不是快速的通用密码哈希函数（例如 SHA256）。截至2018年6月，可以使用的可接受密码哈希算法列表如下：

+   Argon2（在 PHP 7.2 及更新版本中可用）

+   Scrypt

+   **Bcrypt**（PHP 已为您提供；见下文）

+   PBKDF2 与 HMAC-SHA256 或 HMAC-SHA512

幸运的是，现在 PHP 让这一切变得简单。

使用 `password_hash` 哈希密码

在 PHP 5.5 中引入了 `password_hash()`。目前它使用 BCrypt，这是 PHP 支持的最强算法。将来会根据需要更新以支持更多算法，而 `password_compat` 库则提供了 PHP >= 5.3.7 的向前兼容性。

下面我们对一个字符串进行哈希处理，然后将哈希结果与一个新字符串进行比较。因为我们的两个源字符串不同（‘secret-password’ vs. ‘bad-password’），这次登录将失败。

```
<?php
require 'password.php';

$passwordHash = password_hash('secret-password', PASSWORD_DEFAULT);

if (password_verify('bad-password', $passwordHash)) {
    // Correct Password
} else {
    // Wrong password
}
```

`password_hash()` 函数可以为您处理密码的盐值。盐值与算法和“成本”一起存储在哈希的一部分中。`password_verify()` 函数提取这些信息以确定如何检查密码，因此您不需要单独的数据库字段来存储盐值。

## 数据过滤

绝对不要（绝对不要）信任引入到您的 PHP 代码中的外部输入。始终在使用代码之前对外部输入进行过滤和验证。`filter_var()` 和 `filter_input()` 函数可以对文本进行过滤和验证文本格式（例如电子邮件地址）。

外部输入可以是任何东西：`$_GET` 和 `$_POST` 表单输入数据，`$_SERVER` 超全局变量中的一些值，以及通过 `fopen('php://input', 'r')` 获取的 HTTP 请求体。记住，外部输入不仅限于用户提交的表单数据。上传和下载的文件、会话值、Cookie 数据以及来自第三方网络服务的数据也是外部输入。

尽管外部数据可以存储、组合并稍后访问，但它仍然是外部输入。每次在代码中处理、输出、连接或包含数据时，请问自己是否已适当过滤数据并且可以信任它。

根据其用途，数据可能会以不同方式进行*过滤*。例如，当未经过滤的外部输入传递到 HTML 页面输出时，它可以在您的站点上执行 HTML 和 JavaScript！这被称为跨站点脚本（XSS）攻击，可能是非常危险的。避免 XSS 的一种方法是在将用户生成的所有数据输出到页面之前，通过 `strip_tags()` 函数删除 HTML 标签或通过 `htmlentities()` 或 `htmlspecialchars()` 函数转义具有特殊含义的字符为其相应的 HTML 实体。

另一个例子是传递选项以在命令行上执行。这可能非常危险（通常不是一个好主意），但您可以使用内置的 `escapeshellarg()` 函数来对执行的命令参数进行过滤。

最后一个例子是接受外部输入以确定从文件系统加载的文件。这可能被更改为文件路径的文件名利用。您需要从文件路径中删除 `"/"`、`"../"`、[null bytes](https://www.php.net/security.filesystem.nullbytes) 或其他字符，以便不能加载隐藏、非公共或敏感文件。

### 过滤

过滤会删除（或转义）外部输入中的非法或不安全字符。

例如，您应该在将外部输入包含在 HTML 中或将其插入原始 SQL 查询之前对其进行过滤。当您使用 [PDO](#databases) 的绑定参数时，它会为您对输入进行过滤。

有时，在将输入包含在HTML页面中时，需要允许一些安全的HTML标签。这很难做到，许多人通过使用更受限制的格式（如Markdown或BBCode），或者使用像[HTML Purifier](http://htmlpurifier.org/)这样的白名单库来避免这种情况。

[参见净化过滤器](https://www.php.net/filter.filters.sanitize)

### 反序列化

从用户或其他不受信任的来源`unserialize()`数据是危险的。这样做可能允许恶意用户实例化对象（具有用户定义属性），这些对象的析构函数将被执行，**即使对象本身未被使用**。因此，应避免对不受信任的数据进行反序列化。

如果需要向用户传递序列化数据，请使用安全的标准数据交换格式，例如JSON（通过[`json_decode`](https://www.php.net/manual/function.json-decode.php)和[`json_encode`](https://www.php.net/manual/function.json-encode.php)）。

### 验证

验证确保外部输入是您所期望的。例如，在处理注册提交时，您可能需要验证电子邮件地址、电话号码或年龄。

[参见验证过滤器](https://www.php.net/filter.filters.validate)

## 配置文件

创建应用程序配置文件时，最佳实践建议遵循以下方法之一：

+   建议将配置信息存储在无法直接访问的位置，并通过文件系统引入。

+   如果必须将配置文件存储在文档根目录中，请使用`.php`扩展名命名文件。这样即使脚本直接访问，也不会以纯文本输出。

+   配置文件中的信息应根据需要进行保护，可以通过加密或组/用户文件系统权限来实现。

+   建议确保不要将包含敏感信息（如密码或API令牌）的配置文件提交到源代码控制。

## 全局注册变量

**注意：** 从PHP 5.4.0开始，`register_globals`设置已被移除，不再可用。这只是作为警告包含在这里，以提醒正在升级遗留应用程序的用户。

启用`register_globals`配置设置后，应用程序的全局作用域中将可用多种类型的变量（包括来自`$_POST`、`$_GET`和`$_REQUEST`的变量）。这可能会导致安全问题，因为应用程序无法有效判断数据来源。

例如：`$_GET['foo']`可以通过`$foo`访问，这可能会覆盖已声明的变量。

如果您使用PHP < 5.4.0，请确保`register_globals`是**关闭**。

## 错误报告

错误日志可以帮助找到应用程序中的问题点，但也可能将应用程序结构信息暴露给外界。为了有效保护应用程序免受这些消息输出可能引起的问题，您需要在开发与生产（现场）环境中配置服务器。

### 开发

要在**开发**环境中显示每一个可能的错误，请在您的`php.ini`中配置以下设置：

```
display_errors = On
display_startup_errors = On
error_reporting = -1
log_errors = On
```

> 传递值`-1`将显示每一个可能的错误，即使在未来的PHP版本中添加了新级别和常量。从PHP 5.4开始，`E_ALL`常量也是这样。 - [php.net](https://www.php.net/function.error-reporting)

`E_STRICT`错误级别常量在5.3.0中引入，不包括在`E_ALL`中，但在5.4.0中成为`E_ALL`的一部分。这意味着什么？在5.3版本中报告每一个可能的错误意味着您必须使用`-1`或`E_ALL | E_STRICT`。

**按PHP版本报告每一个可能的错误**

+   < 5.3 `-1` 或 `E_ALL`

+   5.3 `-1` 或 `E_ALL | E_STRICT`

+   > 5.3 `-1` 或 `E_ALL`

### 生产环境

要在**生产**环境中隐藏错误，需配置您的`php.ini`如下：

```
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL
log_errors = On
```

在生产环境中使用这些设置，错误仍将被记录到网页服务器的错误日志中，但不会显示给用户。有关这些设置的更多信息，请参阅PHP手册：

# 测试

为您的PHP代码编写自动化测试被认为是最佳实践，可以带来构建良好应用程序的好处。自动化测试是确保在进行更改或添加新功能时应用程序不会中断的强大工具，不应被忽视。

有多种不同类型的PHP测试工具（或框架）可供选择，它们采用不同的方法 - 所有这些方法都试图避免手动测试和大型质量保证团队的需求，以确保最新更改不会破坏现有功能。

## 测试驱动开发

来自[Wikipedia](https://wikipedia.org/wiki/Test-driven_development)：

> 测试驱动开发（TDD）是一种依赖于非常短的开发周期重复的软件开发过程：首先，开发人员编写一个失败的自动化测试用例，定义所需的改进或新功能，然后生成代码以通过该测试，最后将新代码重构为可接受的标准。被誉为开发或“重新发现”该技术的肯特·贝克（Kent Beck）在2003年表示，TDD鼓励简单的设计并增强信心。

您可以为应用程序执行多种不同类型的测试：

### 单元测试

单元测试是一种编程方法，用于确保函数、类和方法在从创建到整个开发周期内的预期工作。通过检查进入和退出各种函数和方法的值，您可以确保内部逻辑的正确性。通过使用依赖注入和构建“模拟”类和存根，您可以验证依赖项是否被正确使用，以实现更好的测试覆盖率。

当您创建一个类或函数时，应为其每个必须具备的行为创建一个单元测试。在非常基本的层面上，您应确保如果发送错误参数则会引发错误，并确保如果发送有效参数则会正常工作。这将有助于确保在开发周期后期对该类或函数进行更改时，旧功能继续按预期工作。唯一的替代方案是在test.php中使用`var_dump()`，这无论是对于构建应用程序（无论大小）都不是一种良好的方式。

单元测试的另一个用途是为开源项目做出贡献。如果您可以编写显示出功能失效（即失败）的测试，然后修复它，并显示测试通过的测试，那么补丁更有可能被接受。如果您运行一个接受拉取请求的项目，则应将此视为要求。

[PHPUnit](https://phpunit.de/) 是编写PHP应用程序单元测试的事实标准测试框架，但也有几个替代方案：

### 集成测试

来自[Wikipedia](https://wikipedia.org/wiki/Integration_testing)：

> 集成测试（有时称为集成与测试，缩写为“I&T”）是软件测试中的一个阶段，其中将单个软件模块组合并作为一个组进行测试。它发生在单元测试之后和验证测试之前。集成测试以经过单元测试的模块作为其输入，将它们组合成更大的聚合体，将在集成测试计划中定义的测试应用于这些聚合体，并以集成系统作为其输出，准备进行系统测试。

许多用于单元测试的同样工具也可以用于集成测试，因为使用了许多相同的原则。

### 功能测试

有时也被称为验收测试，功能测试包括使用工具创建自动化测试，实际使用您的应用程序，而不仅仅是验证单个代码单元的行为是否正确，并确保各个单元能够正确交互。这些工具通常使用真实数据并模拟应用程序的实际用户。

+   [Selenium](https://www.selenium.dev/)

+   [Mink](https://mink.behat.org/)

+   [Codeception](https://codeception.com/) 是一个包含验收测试工具的全栈测试框架。

+   [Storyplayer](https://github.com/MeltwaterArchive/storyplayer) 是一个全栈测试框架，支持按需创建和销毁测试环境。

## 行为驱动开发

有两种不同类型的行为驱动开发（BDD）：SpecBDD 和 StoryBDD。SpecBDD 专注于代码的技术行为，而 StoryBDD 则专注于业务或功能行为或交互。PHP 有适用于两种类型 BDD 的框架。

使用 StoryBDD，您编写可读的人类故事来描述应用的行为。这些故事可以作为实际测试运行。PHP 应用程序中用于 StoryBDD 的框架是[Behat](https://behat.org/)，它灵感来自于 Ruby 的[Cucumber](https://cucumber.io/)项目，并实现了描述功能行为的 Gherkin DSL。

使用 SpecBDD，您编写描述实际代码应如何行为的规范。与测试函数或方法不同，您描述的是函数或方法应如何行为。PHP 提供了[PHPSpec](https://www.phpspec.net/)框架来实现这一目的。此框架灵感来自于 Ruby 的[RSpec 项目](https://rspec.info/)。

### BDD 链接

+   [Behat](https://behat.org/)，这是一个基于 PHP 的 StoryBDD 框架，灵感来自于 Ruby 的[Cucumber](https://cucumber.io/)项目；

+   [PHPSpec](https://www.phpspec.net/)，这是 PHP 的 SpecBDD 框架，灵感来自于 Ruby 的[RSpec](https://rspec.info/)项目；

+   [Codeception](https://codeception.com/) 是一个使用 BDD 原则的全栈测试框架。

# 服务器和部署

PHP 应用程序可以通过多种方式部署和在生产 Web 服务器上运行。

## 平台即服务（PaaS）

PaaS 提供了在网络上运行 PHP 应用所需的系统和网络架构。这意味着几乎不需要为启动 PHP 应用程序或 PHP 框架进行配置。

最近，PaaS 已成为部署、托管和扩展各种规模的 PHP 应用程序的流行方法。您可以在我们的[资源部分](#resources)找到[PHP PaaS“平台即服务”提供商](#php_paas_providers)的列表。

## 虚拟或专用服务器

如果您熟悉系统管理或者有兴趣学习它，虚拟或专用服务器能让您完全控制应用的生产环境。

### nginx 和 PHP-FPM

PHP，通过 PHP 内置的 FastCGI 进程管理器（FPM），与[nginx](https://nginx.org/)非常搭配，后者是一个轻量、高性能的网络服务器。它比 Apache 使用更少的内存，并且能更好地处理更多的并发请求。这在内存不足的虚拟服务器上尤为重要。

### Apache 和 PHP

PHP 和 Apache 有着长久的历史。Apache 高度可配置，有许多可用的[模块](https://httpd.apache.org/docs/2.4/mod/)来扩展功能。它是共享服务器的流行选择，也是 PHP 框架和开源应用程序（如 WordPress）的简易设置。不幸的是，默认情况下，Apache 使用的资源比 nginx 更多，并且无法同时处理那么多访问者。

Apache有几种可能的配置来运行PHP。最常见和最容易设置的是使用[prefork MPM](https://httpd.apache.org/docs/2.4/mod/prefork.html)和`mod_php`。虽然它不是最节省内存的选择，但它是最简单工作和使用的选择。如果你不想深入了解服务器管理方面，这可能是最好的选择。请注意，如果你使用`mod_php`，你必须使用prefork MPM。

或者，如果你想要从Apache中挤出更多性能和稳定性，你可以利用与nginx相同的FPM系统，并使用mod_fastcgi或mod_fcgid运行[worker MPM](https://httpd.apache.org/docs/2.4/mod/worker.html)或[event MPM](https://httpd.apache.org/docs/2.4/mod/event.html)。这种配置在内存效率和速度上显著提高，但设置工作量更大。

如果你运行的是Apache 2.4或更高版本，你可以使用[mod_proxy_fcgi](https://httpd.apache.org/docs/current/mod/mod_proxy_fcgi.html)来获得易于设置的高性能。

## 共享服务器

PHP之所以如此受欢迎，要感谢共享服务器。很难找到没有安装PHP的主机，但务必确保它是最新版本。共享服务器允许你和其他开发者将网站部署到一台机器上。好处是它已经成为一种廉价的商品。缺点是你永远不知道你的邻居租户会制造什么样的骚动：使服务器负载过重或打开安全漏洞是主要问题。如果你的项目预算允许避开共享服务器，那么你应该这样做。

确保你的共享服务器提供最新版本的PHP。

## 构建和部署你的应用程序

如果你发现自己在手动修改数据库模式或在更新文件（手动）之前手动运行测试，请三思！每增加一个手动任务来部署应用的新版本，都会增加出现致命错误的可能性。无论你是在处理简单的更新、全面的构建过程，甚至是连续集成策略，[构建自动化](https://wikipedia.org/wiki/Build_automation)都是你的朋友。

你可能想要自动化的任务包括：

+   依赖管理

+   编译、压缩你的资产

+   运行测试

+   创建文档

+   打包

+   部署

部署工具可以描述为处理软件部署常见任务的一组脚本。部署工具不是你软件的一部分，它是从‘外部’操作你的软件。

有许多开源工具可用于帮助你进行构建自动化和部署，有些是用PHP编写的，有些不是。这不应阻止你使用它们，如果它们更适合特定的工作。这里有一些例子：

[Phing](https://www.phing.info/) 可以通过 XML 构建文件控制您的打包、部署或测试过程。Phing（基于 [Apache Ant](https://ant.apache.org/)）提供了一组丰富的任务，通常用于安装或更新 Web 应用程序，并可以通过编写的 PHP 附加自定义任务进行扩展。它是一个坚固和稳健的工具，并存在已久，但由于处理配置的方式（XML 文件），这个工具可能被视为有些老式。

[Capistrano](https://capistranorb.com/) 是一个面向*中高级程序员*的系统，可以在一个或多个远程机器上以结构化、可重复的方式执行命令。它预配置用于部署 Ruby on Rails 应用程序，但你也可以成功地部署 PHP 系统。成功使用 Capistrano 取决于对 Ruby 和 Rake 的工作知识。

[Ansistrano](https://ansistrano.com) 是一对 Ansible 角色，用于轻松管理脚本应用程序（如 PHP、Python 和 Ruby）的部署过程（部署和回滚）。它是 [Capistrano](https://capistranorb.com/) 的一个 Ansible 移植版。已被许多 PHP 公司广泛使用。

[Deployer](https://deployer.org/) 是一个用 PHP 编写的部署工具。它简单且功能强大，支持并行运行任务、原子部署以及在服务器之间保持一致性。提供了用于 Symfony、Laravel、Zend Framework 和 Yii 的常见任务配方。Younes Rafie 的文章 [Easy Deployment of PHP Applications with Deployer](https://www.sitepoint.com/deploying-php-applications-with-deployer/) 是一个使用该工具部署应用程序的优秀教程。

[Magallanes](https://www.magephp.com/) 是另一个用 PHP 编写的工具，其配置简单地在 YAML 文件中完成。它支持多个服务器和环境、原子部署，并内置了一些可用于常见工具和框架的任务。

#### 进一步阅读：

### 服务器配置

管理和配置服务器在面对多台服务器时可能是一项艰巨的任务。有一些工具可以帮助你自动化基础设施，确保你拥有正确的服务器配置。它们通常与大型云托管提供商（如亚马逊网络服务、Heroku、DigitalOcean等）集成，用于管理实例，从而使应用程序的扩展变得更加容易。

[Ansible](https://www.ansible.com/) 是一种通过 YAML 文件管理基础设施的工具。它易于入门，能够管理复杂和大规模的应用程序。提供了用于管理云实例的 API，并可以通过动态清单使用特定工具进行管理。

[Puppet](https://puppet.com/) 是一个工具，有自己的语言和文件类型用于管理服务器和配置。它可以在主/客户端设置中使用，也可以在“无主”模式下使用。在主/客户端模式中，客户端将定期轮询中央主机以获取新的配置，并在必要时更新自己。在无主模式下，您可以将更改推送到您的节点。

[Chef](https://www.chef.io/) 是一个基于 Ruby 的强大系统集成框架，您可以使用它来构建整个服务器环境或虚拟机。通过他们的服务 OpsWorks，它能够很好地与亚马逊网络服务集成。

#### 进一步阅读：

### 持续集成

> 持续集成是一个软件开发实践，团队成员经常集成他们的工作，通常每个人至少每天集成一次 —— 导致一天内多次集成。许多团队发现这种方法显著减少了集成问题，并允许团队更快速地开发一致的软件。

*– Martin Fowler*

有不同的方法可以为 PHP 实施持续集成。[Travis CI](https://www.travis-ci.com/) 在使得即使对于小型项目也成为可能的持续集成方面做得非常出色。Travis CI 是一个托管的持续集成服务。它可以与 GitHub 集成，并支持包括 PHP 在内的许多语言。GitHub 通过 [GitHub Actions](https://docs.github.com/en/actions) 提供持续集成工作流支持。

#### 进一步阅读：

# 虚拟化

在开发和生产中在不同环境上运行应用程序可能会导致在上线时出现奇怪的错误。当与开发团队一起工作时，保持不同开发环境中使用的所有库的相同版本也很棘手。

如果您在 Windows 上开发并部署到 Linux（或其他非 Windows 系统），或者与团队合作开发，您应考虑使用虚拟机。这听起来有些棘手，但除了广为人知的虚拟化环境如 VMware 或 VirtualBox 外，还有其他工具可以帮助您在几个简单的步骤内设置虚拟环境。

## Vagrant

[Vagrant](https://www.vagrantup.com/) 可以帮助您在已知的虚拟环境之上构建您的虚拟机，并且根据单个配置文件配置这些环境。这些虚拟机可以手动设置，或者您可以使用像[Puppet](https://puppet.com/) 或 [Chef](https://www.chef.io/) 这样的“配置管理”软件来为您完成。配置基础虚拟机是确保多个虚拟机以相同方式设置并消除您维护复杂“设置”命令列表的好方法。您还可以“销毁”基础虚拟机并重新创建，而不需要进行许多手动步骤，这使得创建“全新”安装变得容易。

Vagrant创建用于在主机和虚拟机之间共享代码的文件夹，这意味着您可以在主机上创建和编辑文件，然后在虚拟机中运行代码。

## Docker

[Docker](https://www.docker.com/) - 一个轻量级的替代完整虚拟机的工具 - 之所以被称为“容器”，是因为它专注于“容器”。容器是一个构建块，在最简单的情况下，执行一个特定的任务，例如运行一个Web服务器。一个“镜像”是构建容器所需的软件包 - Docker拥有一个充满这些镜像的仓库。

典型的LAMP应用程序可能包含三个容器：一个Web服务器，一个PHP-FPM进程和MySQL。与Vagrant中的共享文件夹类似，您可以将应用程序文件留在原地，并告诉Docker在哪里找到它们。

您可以通过命令行生成容器（参见下面的示例），或者为您的项目构建一个`docker-compose.yml`文件，指定要创建的容器及其彼此的通信方式，以便于维护。

如果您正在开发多个网站，并希望通过将每个网站安装在自己的虚拟机上来实现分离，但是没有足够的磁盘空间或时间来保持所有内容的最新状态，则Docker可能会有所帮助。它非常高效：安装和下载速度更快，您只需存储每个镜像的一个副本，容器需要更少的RAM并共享同一个操作系统内核，因此可以同时运行更多服务器，并且停止和启动它们仅需几秒钟，无需等待完整的服务器启动。

### 示例：在Docker中运行您的PHP应用程序

在[安装docker](https://docs.docker.com/get-docker/)后，您可以通过一个命令启动一个Web服务器。以下命令将下载一个带有最新PHP版本的完整功能的Apache安装，并映射`/path/to/your/php/files`到文档根目录，您可以在`http://localhost:8080`查看：

```
docker run -d --name my-php-webserver -p 8080:80 -v /path/to/your/php/files:/var/www/html/ php:apache
```

这将初始化并启动您的容器。`-d`使其在后台运行。要停止和启动它，只需运行`docker stop my-php-webserver`和`docker start my-php-webserver`（其他参数不再需要）。

### 了解更多关于Docker的信息

上面的命令显示了运行基本服务器的快速方式。您可以做更多的事情（并且在[Docker Hub](https://hub.docker.com/)中有数千个预构建的镜像）。花些时间学习术语，并阅读[Docker用户指南](https://docs.docker.com/)以获取最佳体验，不要运行未经检查安全性的随机代码 - 非官方镜像可能没有最新的安全补丁。如果有疑问，请坚持使用[官方仓库](https://hub.docker.com/explore/)。

[PHPDocker.io](https://phpdocker.io/) 网站将为您生成所需的所有文件，以便完全功能的LAMP/LEMP堆栈，包括您选择的PHP版本和扩展。

# 缓存

PHP本身速度很快，但当你进行远程连接、加载文件等操作时可能会出现瓶颈。幸运的是，有各种工具可用于加速应用程序的某些部分，或者减少这些各种耗时任务运行次数的工具。

## 操作码缓存通过将操作码存储在内存中并在后续调用中重复使用来防止冗余编译。通常会首先检查文件的签名或修改时间，以防有任何更改。

当执行PHP文件时，必须首先将其编译为[opcodes](https://php-legacy-docs.zend.com/manual/php4/en/internals2.opcodes)（CPU的机器语言指令）。如果源代码未更改，操作码将保持不变，因此这个编译步骤将成为CPU资源的浪费。

操作码缓存

PHP 5.5以后已经内置了一个操作码缓存 - [Zend OPcache](https://www.php.net/book.opcache)。根据你的PHP包/分发版本，它通常默认开启 - 检查[opcache.enable](https://www.php.net/manual/opcache.configuration.php#ini.opcache.enable)以及`phpinfo()`的输出来确认。对于早期版本，有一个PECL扩展。

阅读更多关于操作码缓存：

## 对象缓存

有时缓存单个对象在你的代码中可以带来好处，比如对于获取成本高昂或者数据库调用结果不太可能改变的数据。你可以使用对象缓存软件将这些数据片段保存在内存中，以便以后极快的访问。如果在检索数据后将这些项保存到数据存储中，然后直接从缓存中获取后续请求，你可以显著提高性能，同时减少数据库服务器的负载。

许多流行的字节码缓存解决方案还允许你缓存自定义数据，因此有更多的理由利用它们。APCu和WinCache都提供API，用于将数据从你的PHP代码保存到它们的内存缓存中。

最常用的内存对象缓存系统是APCu和memcached。APCu是对象缓存的优秀选择，它包含一个简单的API，用于将自定义数据添加到其内存缓存中，设置和使用都非常简单。APCu的一个真正限制是它与安装它的服务器绑定。另一方面，memcached作为一个独立的服务安装，并且可以通过网络访问，这意味着你可以将对象存储在一个中心位置的超快速数据存储中，并且许多不同的系统可以从中提取。

注意，在Web服务器内运行PHP作为(Fast-)CGI应用程序时，每个PHP进程都有自己的缓存，即APCu数据不会在工作进程之间共享。在这些情况下，你可能希望考虑使用memcached，因为它不与PHP进程绑定。

在网络配置中，APCu通常在访问速度上优于memcached，但memcached能够更快速和更远程地扩展。如果您不希望在应用程序中运行多个服务器，或者不需要memcached提供的额外功能，则APCu可能是对象缓存的最佳选择。

使用APCu的示例逻辑：

```
<?php
// check if there is data saved as 'expensive_data' in cache
$data = apcu_fetch('expensive_data');
if ($data === false) {
    // data is not in cache; save result of expensive call for later use
    apcu_add('expensive_data', $data = get_expensive_data());
}

print_r($data);
```

注意，在PHP 5.5之前，有APC扩展，它提供了对象缓存和字节码缓存。新的APCu项目旨在将APC的对象缓存引入PHP 5.5+，因为PHP现在有了内置的字节码缓存（OPcache）。

### 了解更多关于流行的对象缓存系统的信息：

## PHPDoc

PHPDoc是用于注释PHP代码的非正式标准。有很多不同的[tags](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/index.html)可用。您可以在[PHPDoc手册](https://docs.phpdoc.org/latest/index.html)中找到完整的标签列表和示例。

下面是一个如何使用几种方法记录类的示例；

```
<?php
/**
 * @author A Name <a.name@example.com>
 * @link https://www.phpdoc.org/docs/latest/index.html
 */
class DateTimeHelper
{
    /**
     * @param mixed $anything Anything that we can convert to a \DateTime object
     *
     * @throws \InvalidArgumentException
     *
     * @return \DateTime
     */
    public function dateTimeFromAnything($anything)
    {
        $type = gettype($anything);

        switch ($type) {
            // Some code that tries to return a \DateTime object
        }

        throw new \InvalidArgumentException(
            "Failed Converting param of type '{$type}' to DateTime object"
        );
    }

    /**
     * @param mixed $date Anything that we can convert to a \DateTime object
     *
     * @return void
     */
    public function printISO8601Date($date)
    {
        echo $this->dateTimeFromAnything($date)->format('c');
    }

    /**
     * @param mixed $date Anything that we can convert to a \DateTime object
     */
    public function printRFC2822Date($date)
    {
        echo $this->dateTimeFromAnything($date)->format('r');
    }
}
```

整个类的文档包含[@author](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/author.html)标签和[@link](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/link.html)标签。[@author](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/author.html)标签用于记录代码的作者，并可以被

在类内部，第一个方法有一个[@param](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/param.html)标签，用于记录传递给方法的参数类型、名称和描述。此外，它还有[@return](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/return.html)和[@throws](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/throws.html)标签，分别用于记录返回类型和可能抛出的异常。

第二和第三个方法非常相似，并且像第一个方法一样有一个[@param](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/param.html)标签。第二个和第三个方法文档块之间的重要区别在于[@return](https://docs.phpdoc.org/latest/guide/references/phpdoc/tags/return.html)标签的包含/排除。`@return void`明确告知我们没有返回值；历史上省略`@return void`语句也会导致相同的（无返回）操作。

## 要关注的人物

当您刚开始时，要找到有趣和有知识的PHP社区成员是很困难的。您可以在以下地方找到一个简略的PHP社区成员列表，帮助您入门：

## 框架

许多 PHP 开发者使用框架来构建 Web 应用程序，而不是重复发明轮子。框架抽象了许多低级关注点，并提供了有助于完成常见任务的易于使用的接口。

并非每个项目都需要使用框架。有时候纯 PHP 就是正确的选择，但如果确实需要框架，那么有三种主要类型可供选择：

+   微框架

+   全栈框架

+   组件框架

微框架本质上是将 HTTP 请求快速路由到回调、控制器、方法等的包装器，并有时附带一些额外的库来辅助开发，比如基本的数据库包装器等。它们主要用于构建远程 HTTP 服务。

许多框架在微框架提供的基础上增加了大量功能；这些被称为全栈框架。它们通常捆绑了 ORM、身份验证包等。

基于组件的框架是专业化和单一用途库的集合。不同的基于组件的框架可以一起使用，以制作微型或全栈框架。

## 组件

如上所述，“组件”是实现创建、分发和实施共享代码的另一种方法。存在各种组件仓库，其中主要的两个是：

这两个仓库都有与之相关的命令行工具，以帮助安装和升级过程，并且在[依赖管理](/#dependency_management)部分有更详细的解释。

还有基于组件的框架和提供完全无框架的组件供应商。这些项目提供了另一个包的来源，理想情况下，这些包几乎不依赖于其他包或特定框架。

例如，你可以使用[FuelPHP Validation package](https://github.com/fuelphp/validation)，而无需使用 FuelPHP 框架本身。

*Laravel 的[Illuminate 组件](https://github.com/illuminate)将更好地与 Laravel 框架解耦。目前，只有与 Laravel 框架最好解耦的组件列在上面。*

## 其他有用资源

### 速查表

### 更多最佳实践

### PHP 和 Web 开发社区的新闻

你可以订阅每周通讯，以保持对新库、最新消息、活动和一般公告的了解，同时也会不时发布其他资源：

你可能对其他平台上的每周更新感兴趣；这里是[一些列表](https://github.com/jondot/awesome-weekly)。

### PHP 宇宙

## 视频教程

### YouTube 频道

### 付费视频

## 书籍

PHP 有很多书籍；可悲的是，有些现在已经相当老旧且不再准确。特别是，避免关于“PHP 6”的书籍，这个版本现在永远不会存在。PHP 5.6 之后的下一个重要版本是“PHP 7”，[部分原因就在于此](https://wiki.php.net/rfc/php6)。

本节旨在成为关于 PHP 开发推荐书籍的活跃文档。如果您希望您的书籍被添加，请发送 PR，我们会审查其相关性。

### 免费书籍

### 付费书籍

+   [PHP & MySQL](https://phpandmysql.com/) - 一本具有出色插图的 PHP 和 MySQL 基础全覆盖书籍，提供实际示例。

+   [构建你不会讨厌的 API](https://apisyouwonthate.com/) - 每个人都想要一个 API，所以你应该学习如何构建它们。

+   [现代 PHP](https://www.oreilly.com/library/view/modern-php/9781491905173/) - 涵盖现代 PHP 的特性、最佳实践、测试、调优、部署以及设置开发环境。

+   [构建安全的 PHP 应用](https://leanpub.com/buildingsecurephpapps) - 学习一个资深开发人员通常在多年经验中获得的安全基础知识，所有内容都被压缩到一个快速且简单的手册中。

+   [现代化 PHP 中的传统应用](https://leanpub.com/mlaphp) - 通过一系列小而具体的步骤控制您的代码。

+   [保护 PHP：核心概念](https://leanpub.com/securingphp-coreconcepts) - 介绍一些最常见的安全术语，并提供日常 PHP 中的一些示例。

+   [扩展 PHP](https://www.scalingphpbook.com/) - 别再当系统管理员，回到编码中来。

+   [PHP 信号处理](https://leanpub.com/signalingphp) - 在编写从命令行运行的 PHP 脚本时，PCNLT 信号非常有帮助。

+   [最小可行测试](https://leanpub.com/minimumviabletests) - PHP 测试倡导者 Chris Hartjes 讲述了他认为您需要了解的最低测试知识。

+   [PHP 中的领域驱动设计](https://leanpub.com/ddd-in-php) - 展示 PHP 编写的真实示例，展示领域驱动设计的架构风格（六边形架构、CQRS 或事件溯源）、战术设计模式以及边界上下文集成。

## PHP 用户组

如果您住在一个大城市，很可能附近有一个 PHP 用户组。您可以在 [PHP.ug](https://php.ug/) 找到您的本地 PUG。也可以通过 [Meetup.com](https://www.meetup.com/find/) 或使用您喜欢的搜索引擎（如 [Google](https://www.google.com/search?q=php+user+group+near+me)）搜索 `php user group near me`。如果您住在一个较小的城镇，可能没有本地的 PUG；如果是这样，请自己创建一个！

特别提到两个全球性用户组：[NomadPHP](https://nomadphp.com/) 和 [PHPWomen](https://twitter.com/PHPWomen)。[NomadPHP](https://nomadphp.com/) 每月两次在线用户组会议，由 PHP 社区的顶级演讲者进行演讲。[PHPWomen](https://twitter.com/PHPWomen) 是一个最初面向 PHP 世界中的女性的非独占性用户组。会员对支持更多样化的社区的人士开放。PHPWomen 提供支持、导师制和教育网络，并且普遍促进创建“女性友好”和专业氛围。

[在 PHP Wiki 上了解用户组](https://wiki.php.net/usergroups)

## PHP 会议

PHP 社区还在世界许多国家举办更大的区域和国家性会议。PHP 社区的知名成员通常在这些大型活动中演讲，因此这是直接从行业领袖学习的绝佳机会。

[寻找 PHP 会议](https://www.php.net/conferences/index.php)

## ElePHPants

[ElePHPant](https://www.php.net/elephpant.php) 是 PHP 项目中设计有大象的美丽吉祥物。它最初由 [Vincent Pontier](http://www.elroubio.net/) 设计于 1998 年，是成千上万只 elePHPant 的精神父亲 - 十年后，可爱的毛绒大象玩具也应运而生。现在，elePHPants 在许多 PHP 会议上和许多 PHP 开发者的计算机上出现，以供娱乐和灵感。

[与 Vincent Pontier 的采访](https://7php.com/elephpant/)
