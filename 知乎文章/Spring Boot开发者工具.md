Spring Boot 1.3引入了一组新的开发者工具，可以让你在开发时更方便地使用Spring Boot， 包括如下功能。

- 自动重启:当Classpath里的文件发生变化时，自动重启运行中的应用程序。 
-  LiveReload支持:对资源的修改自动触发浏览器刷新。
-  远程开发:远程部署时支持自动重启和LiveReload。
- 默认的开发时属性值:为一些属性提供有意义的默认开发时属性值。



Spring Boot的开发者工具采取了库的形式，可以作为依赖加入项目。如果你使用Gradle来构 建项目，可以像下面这样在build.gradle文件里添加开发工具:

compile "org.springframework.boot:spring-boot-devtools"

在Maven POM里添加<dependency>是这样的:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

当应用程序以完整打包好的JAR或WAR文件形式运行时，开发者工具会被禁用，所以没有必 要在构建生产部署包前移除这个依赖。

### 1. 自动重启

​		在激活了开发者工具后，Classpath里对文件做任何修改都会触发应用程序重启。为了让重启 速度够快，不会修改的类(比如第三方JAR文件里的类)都加载到了基础类加载器里，而应用程 序的代码则会加载到一个单独的重启类加载器里。检测到变更时，只有重启类加载器重启。

有些Classpath里的资源变更后不需要重启应用程序。像Thymeleaf这样的视图模板可以直接 编辑，不用重启应用程序。在/static或/public里的静态资源也不用重启应用程序，所以Spring Boot 开发者工具会在重启时排除掉如下目录:/META-INF/resources、/resources、/static、/public和 /templates。

可以设置spring.devtools.restart.exclude属性来覆盖默认的重启排除目录。例如，你只排除/static和/templates目录，可以像这样设置spring.devtools.restart. exclude:

```yaml
    spring:
      devtools:
        restart:
          exclude: /static/**,/templates/**
```

另一方面，如果想彻底关闭自动重启，可以将spring.devtools.restart.enabled设置 为false:

```yaml
    spring:
      devtools:
        restart:
          enabled: false
```

另外，还可以设置一个触发文件，必须修改这个文件才能触发重启。例如，在修改名为.trigger 的文件前你都不希望执行重启，那么你只需像这样设置spring.devtools.restart.trigger- file属性:

```yaml
    spring:
      devtools:
        restart:
          trigger-file: .trigger
```

如果你的IDE会连续编译修改的文件，那触发文件还是很有用的。没有触发文件的话，每次 变更都会触发重启。有触发文件，就能保证只有你想重启时才会发生重启(修改触发文件即可)。

### 2. LiveReload

在Web应用程序开发过程中，最常见的步骤大致如下。
 (1) 修改要呈现的内容(比如图片、样式表、模板)。
 (2) 点击浏览器里的刷新按钮，查看修改的结果。
 (3) 回到第1步。
 虽然这并不难，但如果能不点刷新就直接看到修改结果，那岂不是更好?
 Spring Boot的开发者工具集成了LiveReload(http://livereload.com)，可以消除刷新的步骤。

激活开发者工具后，Spring Boot会启动一个内嵌的LiveReload服务器，在资源文件变化时会触发 浏览器刷新。你要做的就是在浏览器里安装LiveReload插件。

如 果 想 要 禁 用 内 嵌 的 LiveReload 服 务 器 ， 可 以 将 spring.devtools.livereload. enabled设置为false:

```yaml
    spring:
      devtools:
        livereload:
          enabled: false
```

### 3. 远程开发

​		 这里后面我会单独些一篇IDEA里的实践文章 @todo

### 4. 默认的开发时属性

​      有些配置属性通常在开发时设置，从来不用在生产环境里。比如视图模板缓存，在开发时最 好关掉，这样你可以立刻看到修改的结果。但在生产环境里，为了追求更好的性能，应该开启视 图模版缓存。

默认情况下，Spring Boot会为其支持的各种视图模板(Thymeleaf、Freemarker、Velocity、 Mustache和Groovy模板)开启缓存选项。但如果存在Spring Boot的开发者工具，这些缓存就会 禁用。

实际上，这就是说在开发者工具激活后，如下属性会设置为false。

-  spring.thymeleaf.cache
- spring.freemarker.cache
- spring.velocity.cache

- spring.mustache.cache
- spring.groovy.template.cache

这样一来，就不用在开发时(在一个开发时使用的Profile配置里)禁用它们了。



### 5.全局配置开发者工具

​      你应该已经注意到了，在使用开发者工具时，你通常会在多个项目里使用相同的设置。

举个 例子，如果你使用了重启触发文件，那么你很可能在多个项目里都使用相同的触发文件名。

相比 在每个项目里重复开发者工具配置，对开发者工具做全局配置显得更方便一些。

要实现这个目的，可以在你的主目录(home directory)里创建一个名为.spring-boot-devtools. properties的文件。

(请注意，文件名用“.”开头。)在那个文件里，你可以设置希望在多个项目 里共享的各种开发者工具属性。

例如，假设你想把触发文件的名称设置为.trigger，在所有Spring Boot项目里禁用LiveReload。

 你可以创建一个.spring-boot-devtools.properties文件，包含如下内容:

```
    spring.devtools.restart.trigger-file=.trigger
    spring.devtools.livereload.enabled=false
```

要是你想覆盖这些配置，可以在每个项目的application.properties或application.yml文件里设置 特定于每个项目的属性。