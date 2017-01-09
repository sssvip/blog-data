date: 2017/01/03
title: JAVA后端工作流推荐一--基于SpringBoot快速项目构建
tags: 
- springboot
- gradle
- spring
- starter
---
更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](2017/01/09/workflow/)

#### 此文阅读完效果:

1. 对SpringBoot极其部分依赖有初步认识
2. 对gradle有初步认识
3. 基于[https://start.spring.io/](https://start.spring.io/)构建出项目初始文件，不用再为环境搭建而发愁

<!-- more -->
### SpringBoot 是什么？
Spring Boot takes an opinionated view of building production-ready Spring applications. Favors convention over configuration and is designed to get you up and running as quickly as possible.


上面的这段wiki来自[SpringBoot项目Wiki](https://github.com/spring-projects/spring-boot/wiki),SpringBoot基于约定优于配置原则，能帮我们更快构建Spring应用，不用再配置繁琐的xml文件，更多关于SpringBoot的介绍可以了解[SpringBoot官网](http://projects.spring.io/spring-boot/)。

### 如何构建SpringBoot项目？
**方式一--手动添加**：

正如[SpringBoot官网](http://projects.spring.io/spring-boot/)介绍的Quick Start一样,选择我们常用的构建工具，这里我选择了Gradle，你同样可以选择Maven方式，道理一样。

![quickstart](https://sssvip.github.io/img/springboot/quickstart.png)

将compile("org.springframework.boot:spring-boot-starter-web:1.4.3.RELEASE")添加进gradle构建文件build.gradle中dependencies项中即可，如果你还不熟悉gradle的使用，可以查看[Gradle官网](https://gradle.org/)

**方式二--SPRING INITIALIZR(推荐)**：

访问Spring官方提供的[SPRING INITIALIZR](https://start.spring.io/)如图所示：
![springinitializr](https://sssvip.github.io/img/springboot/springinitializr.png)

**详细步骤：**

1. **根据项目选型选择构建工具Gradle或Maven,这里我选择了`Gradle`**
2. **版本号一般就用其默认的最新发布版，这里默认是`1.4.3`**
3. **输入组织唯一标识Group,这里我输入了`com.minixiao`**
4. **输入项目唯一标识Artifact,这里我输入了`helloworld`**
5. **在Search for dependencies可以输入需要依赖的项目，然后选择添加**
	![springinitializr-search-for-dependencies](https://sssvip.github.io/img/springboot/springinitializr-search-for-dependencies.png)
	![springinitializr-search-for-dependencies-selected](https://sssvip.github.io/img/springboot/springinitializr-search-for-dependencies-selected.png)
6. **推荐点击`Switch to the full version`，可以更方便的看到全部SpringBoot官方提供的Starter.**

	这里你点击`Switch to the full version`后可以看到，我们可以选择Packaging方式，和Java Version(我们采用JDK1.8),因为这些数据都会最后生成到build.gradle中去，推荐项目中Packaging采用Jar的打包方式，这样因为在打包是会自动将所需要的依赖一起打进jar包，最后项目构建产生的jar包只需要在装有JRE环境上的计算机上就能运行，无需再下载SpringBoot相关依赖或者是其他项目中引入的依赖，这也是SpringBoot官方推荐的fat jar打包方式，一次打包到处运行。
	![switch-full-version](https://sssvip.github.io/img/springboot/switch-full-version.png)
	![switch-full-version-dependencies](https://sssvip.github.io/img/springboot/switch-full-version-dependencies.png)
	后面您就能看见很多多选项，这就能看到很多SpringBoot官方提供的Starter,我们根据项目中用到的技术选择即可，这里简单介绍几个常用的选项。

	1. **Web** `Full-stack web development with Tomcat and Spring MVC`

		> 其实这里的英文说明已经说明的很清楚了“全栈式的WEB开发，集成了Tomcat和Spring MVC”,我们在项目中直接可以使用SpringMVC相关技术，直接用就行，很多配置都不需要，Tomcat就自动打包进我们的jar包了，直接有了内置的Tomcat容器，运行的计算机上只需要JRE环境，而不需要再安装jetty或者tomcat这种web容器，这也是SpringBoot的一大特色，简化配置，更多的关注业务开发，快速构建和运行。

	2. **Thymeleaf** `Thymeleaf templating engine, including integration with Spring`

		> 模版引擎，自动和Spring集成。同类模版引擎还有Freemarker，Groovy Templates等等，我们团队内推荐用Thymeleaf，在前后端未完全分离的情况下，Thymeleaf模版技术可以很好的保留前端原样，前后端协作在某种程度上说是很方便的，更多用法查看[thymeleaf官网](http://thymeleaf.org/)。
	
	3. **JPA** `Java Persistence API including spring-data-jpa, spring-orm and Hibernate`

		> 包含了spring-data-jpa, spring-orm and Hibernate，这位面向对象开发来说节省了不少成本，不过要高效率的使用，还需要多研读[spring-data-jpa官方文档](http://docs.spring.io/spring-data/jpa/docs/1.10.4.RELEASE/reference/html/#repositories.special-parameters)和[Hibernate文档](http://hibernate.org/orm/documentation/5.2/)

	4. **PostgreSQL** `PostgreSQL jdbc driver`

		> PostgreSQL 驱动是少不了的，根据项目需要可以选择Mysql等等其他数据库驱动。
	
	5. **其他依赖**

		> 根据项目需要，按需选择，具体到对应的依赖，可以看其注释说明就能知道其作用，按需引用即可。

7. **下载生成构建的项目**
	
	在当前网页点击绿色的`Generate Project`按钮或者敲`alt+enter`键可自动下载刚产生的项目初始文件，如果不出意外你已经拿到了由项目唯一标识Artifact内容命名的zip压缩文件，当然此次下载的即是`helloword.zip`
	![helloworld-zip](https://sssvip.github.io/img/springboot/helloworld-zip.png)
	
	解压后用tree命令查看目录树结构如下：
	![helloworld-zip-tree](https://sssvip.github.io/img/springboot/helloworld-zip-tree.png)
	
	**目录简单解析**：

	* `.gitignore` git版本控制忽略说明文件，更具项目可自行添加忽略目录，生成的gitignore已满足大多数场景
	* `build.gradle` 在整个构建工具中常用的文件，就像maven中pom.xml的地位一样，可以引入各种插件，各种依赖，设置依赖版本，介入构建过程等等的东西。
	* `gradlew` 这个文件很多时候我们很少关注，你用文本编辑其打开就能看到这样一样注释：`Gradle start up script for UN*X`，这就是Gradle初始化脚本，设置一些环境等等。
	* `gradlew.bat` 这个文件很多时候我们很少关注，你用文本编辑其打开就能看到这样一样注释：`Gradle startup script for Windows`，这你就懂了吧？和gradlew对应。
	* `gradle文件夹` 主要是wrapper文件夹下的gradle-wrapper.properties和gradle-wrapper.jar，作用是达到团队中其他人未安装会自动进行安装gradle等等，具体可以上官网查查。
	* `src文件夹` 主要代码放置地，包含main和test文件夹，这我猜你也懂了。还值得一说的是main文件夹下的resources文件夹，下面存在**application.properties**文件，这个文件是springboot的配置文件，配置相当建档，比以前的xml配置方式简单许多，当然你还可以不用properties格式配置，还可以以yml格式配置，配置可读性更高，配置方式可以自行搜索，这不是在此说明的重点。

	项目初始文件就构建好啦，欢迎继续关注下一篇[项目导入及初步介绍](#)


**提示**：如果本地未使用过springboot 1.4.3.RELEASE初次构建需要等待下载依赖的jar包。

**再说一句**：更多的SpringBoot的使用可以查看[SpringBoot官方文档](http://docs.spring.io/spring-boot/docs/1.4.3.RELEASE/reference/htmlsingle/)，说明非常的详细，建议通篇浏览，解决问题更快捷高效。