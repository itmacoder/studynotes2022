本文主要介绍springboot的其他玩法

### Spring Boot CLI

> Spring Boot CLI是Spring Boot的非必要组成部分。虽然它为Spring带来了惊人的力量，大大 简化了开发，但也引入了一套不太常规的开发模型。

#### 安装 Spring Boot CLI

1. 手工安装Spring Boot CLI

   > 安装Spring Boot CLI最直接的方法大约是下载、解压，随后将它的bin目录添加到系统路径里。
   >
   >  你可以从以下两个地址下载分发包:
   >
   > - http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.3.0.RELEASE/spring- boot-cli-1.3.0.RELEASE-bin.zip
   >
   > -  http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.3.0.RELEASE/spring- boot-cli-1.3.0.RELEASE-bin.tar.gz
   >
   >   下载完成之后，把它解压到文件系统的任意目录里。在解压后的目录里，你会找到一个bin 目录，其中包含了一个spring.bat脚本(用于Windows环境)和一个spring脚本(用于Unix环境)。 把这个bin目录添加到系统路径里，然后就能使用Spring Boot CLI了。

2. 使用Software Development Kit Manager进行安装

软件开发工具管理包(Software Development Kit Manager，SDKMAN，曾用简称GVM)也能用来安装和管理多版本Spring Boot CLI。使用前，你需要先从http://sdkman.io获取并安装SDKMAN。最简单的安装方式是使用命令行:

``` shell
$ curl -s get.sdkman.io | bash
```

跟随输出的指示就能完成SDKMAN的安装。在我的机器上，我在命令行里执行了如下命令:

``` shell
$ source "/Users/mawh/.sdkman/bin/sdkman-init.sh"
```

注意，用户不同，这条命令也会有所不同。我的用户目录是/Users/mawh，因此这也是shell 脚本的根路径。你需要根据实际情况稍作调整。一旦安装好了SDKMAN，就可以用下面的方式 来安装Spring Boot CLI了:

```shell
   $ sdk install springboot
   $ spring --version
   
   // 如果需要升级可以参考以下命令
   $ sdk list springboot //查看版本
   $ sdk default springboot 1.3.0.RELEASE //指定版本安装
```

3. 使用Homebrew进行安装 如果你也是mac系统，那就和我使用的一样了

   > 如果要在OS X的机器上进行开发，你还可以用Homebrew来安装Spring Boot CLI。Homebrew 是OS X的包管理器，用于安装多种不同应用程序和工具。要安装Homebrew，最简单的方法就是 运行安装用的Ruby脚本:
   >
   > ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/ master/install)"
   >
   > 你可以在http://brew.sh看到更多关于Homebrew的内容(还有安装方法)。

要用Homebrew来安装Spring Boot CLI

> tap是向Homebrew添加额外仓库的一种途径。Pivotal是Spring及Spring Boot背后的公司，通过它的tap可以安装Spring Boot。

``` shell
$ brew tap spring-io/tap 
$ brew install spring-boot
```

⚠️⚠️⚠️ 下面这种方式已经过时了
~~$ brew tap pivotal/tap~~
~~$ brew install springboot~~

``` shell
Error: To upgrade Spring Boot, retap it with:
      brew tap spring-io/tap
      brew uninstall springboot
      brew install spring-boot
```



你会看到这样的界面

``` shell
(base) x xMacBook-Pro ~ % brew tap spring-io/tap
==> Tapping spring-io/tap
Cloning into '/usr/local/Homebrew/Library/Taps/spring-io/homebrew-tap'...
remote: Enumerating objects: 327, done.
remote: Counting objects: 100% (327/327), done.
remote: Compressing objects: 100% (138/138), done.
remote: Total 327 (delta 102), reused 326 (delta 101), pack-reused 0
Receiving objects: 100% (327/327), 47.58 KiB | 234.00 KiB/s, done.
Resolving deltas: 100% (102/102), done.
Tapped 1 formula (13 files, 61.9KB).
```

 这个命令比较慢，你需要有个“梯子”,失败了就再试两次

```shell
(base) xx@mawhdeMacBook-Pro ~ % brew install spring-boot
==> Downloading https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.6.5/spring-boot-cli-2.6.5-bin.tar.gz
######################################################################## 100.0%
==> Installing spring-boot from spring-io/tap
==> Caveats
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/spring-boot/2.6.5: 7 files, 15.9MB, built in 2 seconds
```



Homebrew会把Spring Boot CLI安装到/usr/local/bin，之后可以直接使用。可以通过检查版本 号来验证安装是否成功:

```shell
    $ spring --version
```

```shell
(base) xxx@mawhdeMacBook-Pro ~ % spring --version
Spring CLI v2.6.5
```



开启命令行补全 

假如你和我一样是mac用Homebrew安装Spring Boot CLI， 恭喜你命令行补全已经安装完毕。

#### 用 Spring Initializr 初始化 Spring Boot 项目

Spring Initializr有几种用法。 

- 通过Web界面使用。
- 通过Spring Tool Suite使用。 
- 通过IntelliJ IDEA使用。
- 使用Spring Boot CLI使用。

相信大家平时都用过web方式和idea的方式，这里我们重点讲Spring Boot CLI的使用，

如果你想仅仅写代码就完成Spring应用程序的开发，那么Spring Boot CLI是个 不错的选择。然而，Spring Boot CLI的功能还不限于此，它有一些命令可以帮你使用Initializr，通 过它上手开发更传统的Java项目。

Spring Boot CLI包含了一个init命令，可以作为Initializr的客户端界面。init命令最简单的 用法就是创建Spring Boot项目的基线:

``` shell
$ spring init
```

在和Initializr的Web应用程序通信后，init命令会下载一个demo.zip文件。解压后你会看到 一个典型的项目结构，包含一个Maven的pom.xml构建描述文件。Maven的构建说明只包含最基本 的内容，即只有Spring Boot基线和测试起步依赖。

你可能会想要更多的东西。

假设你想要构建一个Web应用程序，其中使用JPA实现数据持久化，使用Spring Security进行 安全加固，可以用--dependencies或-d来指定那些初始依赖:

```shell
    $ spring init -dweb,jpa,security
```

这条命令会下载一个demo.zip文件，包含与之前一样的项目结构，但在pom.xml里增加了 Spring Boot的Web、jpa和security起步依赖。请注意，在-d和依赖之间不能加空格，否则就变成 了下载一个ZIP文件，文件名是web,jpa,security。

默认情况下，无论是Maven的构建说明会产生一个可执行JAR文件。但如果你 想要一个WAR文件，那么可以通过--packaging或者-p参数进行说明:

```shell
$ spring init -dweb,jpa,security --build gradle -p war
```

到目前为止，init命令只用来下载ZIP文件。如果你想让CLI帮你解压那个ZIP文件，可以指 定一个用于解压的目录:

```shell
    $ spring init -dweb,jpa,security --build gradle -p war myapp
```

此处的最后一个参数说明你希望把项目解压到myapp目录里去。

此外，如果你希望CLI把生成的项目解压到当前目录，可以使用--extract或者-x参数:

```shell
$ spring init -dweb,jpa,security --build gradle -p jar -x
```

init命令还有不少其他参数，包括基于Groovy构建项目的参数、指定用Java版本编译的参数，

还有选择构建依赖的Spring Boot版本的参数。可以通过help命令了解所有参数的情况: 

```shell
$ spring help init
```



你也可以查看那些参数都有哪些可选项，为init命令带上--list或-l参数就可以了: 

```shell
$ spring init -l
```

