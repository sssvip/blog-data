date: 2017/1/10
title: JAVA后端工作流推荐系列---目录导航
tags: 
- java
- backend
- workflow
- gitlab
- springboot
- gradle
- docker
- IDEA
- Git
---

JAVA后端工作流推荐系列目录导航

#### 此文阅读完效果:

1. 熟悉推荐的JAVA后端工作流
2. 引发对后端工作流的思考
3. JAVA后端相关基础技术

<!-- more -->

不可避免的肯定存在不合理，疏漏的地方，欢迎交流

工作流方式不唯一，仅作推荐，如有更好更合理的建议，欢迎交流，共同完善我们的开发方式，提高我们的生产力。
 
### 基础环境介绍
> 下面罗列版本为目前使用版本，可根据项目和自己情况与时俱进，尽可能的更新

* **编译环境**：JDK 1.8.0_111
* **版本控制**：Git 2.7.4
* **构建工具**：Gradle 3.2.1
* **仓库管理**：Gitlab 8.14.4
* **持续集成**: Gitlab-Runner、TeamCity
* **IDE工具**：IntelliJ IDEA Community
* **CheckStyle**: 7.3
* **微服务框架**：SpringBoot 1.4.2.RELEASE
* **容器环境**：Docker version 1.12.5, build 7392c3b

### 环境安装
1. [JDK安装](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
> 安装过请忽略，未安装请安装好并设置好环境变量等等必要配置。
2. [IDEA安装](https://www.jetbrains.com/idea/#chooseYourEdition)
> 安装过请忽略，选择Community版本进行下载安装即可，这个应该不会产生问题。

### 项目工作流介绍
1. [SpringBoot项目构建](/2017/01/03/springboot/)
	* SpringBoot 是什么？
	* 如何构建SpringBoot项目？
		* 方式一--手动添加
		* 方式二--SPRING INITIALIZR(推荐)
2. [Gitlab创建项目并导入IDE](/2017/01/04/ide-import-project/)
	* 创建Gitlab项目仓库
		* 创建帐号并登录
		* 创建helloworld项目
	* 从远端仓库Clone克隆至本地
		* Git安装
		* 克隆至本地
	* 添加[上一步](/2017/01/03/springboot/)构建的项目初始文件至仓库，进行版本管理
	* 从本地仓库push推至远端仓库
	* 导入IDE(IntelliJ IDEA)
3. [IDEA插件安装及使用](/2017/01/05/idea-plugins/)
	* Markdown插件安装--Markdown Navigator、Markdown support
	* CheckStyle插件安装
	* FindBugs插件安装
	* 添加CheckStyle的xml配置文件
	* 基于CheckStyle的Formatter导入
4. [Gitlab中Wiki文档同项目空间管理](/2017/01/05/wiki-manager/)
	* Wiki目录初始化
	* Wiki克隆至本地
	* 添加Wiki并push至Gitlab的Wiki
5. [Gitlab Runner对SpringBoot应用进行持续集成--理论篇](/2017/01/09/gitlab-runner/)
	* 了解Gitlab Runner是什么,为什么选择Gitlab Runner
	* 如何安装Gitlab Runner(Docker方式)
	* 如何为项目仓库注册Gitlab Runner
	* 如何选择Gitlab Runner的executor
6. [Gitlab Runner在Gradle构建项目中的应用--实战篇](/2017/01/09/gitlab-runner-gradle/)
	* 添加Checkstyle服务，采用gradle插件方式进行
	* 添加`.gitlab-ci.yml`文件意味着申明进行持续集成
	* 注册Runner
	* 测试效果
	* Gitlab CI简单小结	

原文地址: [https://blog.dxscx.com/2017/01/10/workflow/](https://blog.dxscx.com/2017/01/10/workflow/)
