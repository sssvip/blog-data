date: 2017/1/9
title: JAVA后端工作流推荐五--Gitlab Runner对Gradle构建的SpringBoot项目进行持续集成--理论篇
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

更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](/2017/01/09/workflow/)

目前进度：实现代码静态分析，单元测试，自动构建，自动发布测试环境等

后续期望：实现自动发布测试环境

#### 此文阅读完效果:

1. 了解Gitlab Runner是什么,为什么选择Gitlab Runner
2. 如何安装Gitlab Runner(选用Docker方式)
3. 如何为项目仓库注册Gitlab Runner
4. 如何选择Gitlab Runner的executor

<!-- more -->

当你阅读到这里，你肯定是想基于Gitlab做持续集成，当初最开始准备考虑用Gitlab持续集成的时候，看到了Gitlab CI关键字，强烈建议你阅读一下[Get started with GitLab CI](http://docs.gitlab.com/ce/ci/quick_start/README.html),哪怕不全文看完，就看图片都足矣，你就能了解个大概，这篇文章是在做什么，我们想要什么效果。

#### 使用背景
> 项目的开发初期我们很难避免的是我们需要手动的进行代码静态分析，单元测试，发布测试环境等等一系列流程，由于项目中使用Gitlab进行仓库管理，当然我们可以自己通过webhook进行对clone仓库，代码静态分析，单元测试，自动构建等流程，虽然不是特别复杂，但要专心研究一个CI过程，要保证低错误率和高效率的话还是需要花大量时间和精力的，后面想一想没有必要再次造轮子，现在还没到那个程度，故寻求了很多Gitlab CI(Continuous integration)相关技术现有方案，包括不限于`JetBrains TeamCity CI`、`JIRA`、`Atlassian Bamboo CI`等等，当然JIRA并非持续集成，属于issue tracker,在研究CI的同时对其进行了尝试。后面尝试这些三方CI(对于Gitlab来说)，不是软件版权费用昂贵就是不是特方便用，后面再经几分研究，最后选择了官方的Gitlab Runner,猜想其肯定和Gitlab契合度是非常高的，后续更新和支持也不用担心，更何况可以自主搭建Gitlab Runner，免去版权和收费问题。



### Gitlab Runner是什么
> GitLab Runner is the open source project that is used to run your jobs and send the results back to GitLab. It is used in conjunction with GitLab CI, the open-source continuous integration service included with GitLab that coordinates the jobs.

[官方文档](https://docs.gitlab.com/runner/)对其的解释我觉得是很好的认识Gitlab Runner的方式之一，当你在逐渐使用后可能会衍生其他理解。简单说明Gitlab Runner就是一个执行器(Runner)，结合Gitlab CI执行任务，将任务的结果反馈给Gitlab,达到对项目进行持续集成(包括不限于代码静态分析，单元测试，自动构建，自动发布)的效果。

当我们Gitlab项目仓库有Push请求时会自动触发Gitlab CI,达到触发Gitlab Runner执行我们针对项目的预期的CI(不同项目CI流程可能不一样，后面会说明)流程，并将执行的结果返回给Gitlab，达到对Gitlab项目仓库本次版本变化的`持续集成`同时做到`监控和反馈`。

Gitlab Runner提供了什么服务也就是帮我们做了什么工作才是我们`最关心`的地方：

1. **我们不再用写Webhooks**
> 不用写一个Web应用服务来监控仓库变化，不用在监控到其有版本变化后进行clone或则pull至某个地方再对代码进行检测(当然写这个也是可行的，毕竟Gitlab提供了Webhooks服务，只是在考虑团队情况和公司情况没必要花时间经历去做这一块)
2. **我们不用再关心Runner执行队列问题**
> Gitlab Runner是可共享的，多项目，多任务同时等等特征，我们如果自己写持续集成这一块服务，这一块业务的可用性都是一个问题，更不必说高可用性。
3. **安装、部署、运行方式多样化**
> Gitlab Runner提供的安装、部署、运行多样化服务是非常方便的，如果我们自己写这一块服务，难免符合团队中每一个人的技术栈，用熟悉的技术栈去做也许来的更快，可控性更高。
4. **其他**
> 可从Gitlab Runner的特征去查看选择，为我们确实简化了很多过程，对于更加专注业务的研发有很大的帮助。

### 如何安装Gitlab Runner(Docker方式)
> GitLab Runner can be installed and used on GNU/Linux, macOS, FreeBSD and Windows. You can install it Using Docker, download the binary manually or use the repository for rpm/deb packages that GitLab offers. Below you can find information on the different installation methods:

* Install using GitLab's repository for Debian/Ubuntu/CentOS/RedHat (preferred)
* Install on GNU/Linux manually (advanced)
* Install on macOS (preferred)
* Install on Windows (preferred)
* Install as a Docker Service
* Install in Auto-scaling mode using Docker machine
* Install on FreeBSD
* Install on Kubernetes
* Install the nightly binary manually (development)

可以看到，Gitlab Runner的安装方式是多样化可选的，这里我想我没必要再将这个安装过程详详细细的翻译一遍，这是没有意义的，不能达到安装方式于官方文档的同步更新,更多的查看[Install GitLab Runner文档](https://docs.gitlab.com/runner/#install-gitlab-runner)就行，里面的文档是非常详细的。

难道Docker方式安装Gitlab Runner就说完了？当然不是，虽然我基于官方的安装方式进行了尝试，但是我后面未选择采用官方的Gitlab Runner,因为我们项目中需要JDK,和Gradle的环境，所以需要基于官方Gitlab Runner的镜像进行定制，制作自己团队中环境需要的Docker image,所以我们需要在其基础上加上JDK1.8和Gradle构建工具等等东西，这个方式有很多，基于Docker制定镜像的方式常用的有写Dockerfile和启动基础镜像的容器，进行手动安装后进行build,然而我都没有选择，因为在安全和可用的情况下采用现有的东西就够啦，不必要在目前去花过多时间构建image,所以我选择了上[https://hub.docker.com/](https://hub.docker.com/)选择我需要的镜像，我猜都已经有现有的。

#### 我搜寻这个镜像流程大致是这样的：

1. 前期google，Stack Overflow各种搜索解决方案

2. 打开[https://hub.docker.com/](https://hub.docker.com/)

3. 搜索栏键入搜索关键字`gitlab runner gradle java`

4. 经过几番搜寻，找到了`startext`制作的[gitlab-runner-gradle](https://github.com/startext/gitlab-runner-gradle)

	> gitlab-runner-gradle这个镜像拉去率挺高的，看样子都不错，仔细看gitlab-runner-gradle中包含JDK1.8环境，最后找到他的github地址，[https://github.com/startext/gitlab-runner-gradle](https://github.com/startext/gitlab-runner-gradle)，其中有Dockerfile,想自己通过Dockerfile进行定制的可以参考其写法，不是特难，但要小心，脚本出错率还是较高的

#### 简单的说明下安装包含gradle和jdk环境的Gitlab Runner过程：

> 这里假定你是需要含gradle和jdk环境的Gitlab Runner，如果是其他环境按照上面找寻镜像方式找镜像或者自己制作Docker镜像即可，再假定你的已经安装好Docker环境，没安装也不担心，Docker官方文档查一查几条命令搞定。

1. 拉取镜像

	> docker pull startext/gitlab-runner-gradle:latest

2. 启动镜像

	> docker run -d --name gitlab-runner --restart always v /srv/gitlab-runner/config:/etc/gitlab-runner startext/gitlab-runner-gradle:latest

	这里要说明的是这里的Docker容器挂载路径使用的是[Docker Runner官方安装文档](https://docs.gitlab.com/runner/install/docker.html)中默认的，我觉得这种方式的好处是不用自己去记忆它，方便管理。

到这里我猜你已经安装好并运行，没运行成功也不用担心，再次查看[Gitlab Runner官方文档](https://docs.gitlab.com/runner/#install-gitlab-runner)即可。

### 如何为项目仓库注册Gitlab Runner

1. 为什么要给项目仓库注册Gitlab Runner?
	
	文章你已经看到了这里，我们希望有共有的知识是我们想将项目进行持续集成，那么我们对`项目简单假设`：

	1. 我们在`Gitlab`中有A-JAVA(需要main方法运行服务的)项目，B-JAVA基础工具(静态工具，不需要运行)项目，C-文档项目等等一系列项目
	2. A项目需要进行全局Java代码静态分析(包括不限于checkstyle和findbugs)，单元测试，构建jar包或war包什么的。
	3. B项目只需要进行代码静态分析
	4. C项目进行文档格式语法分析

	基于你上文安装Gitlab Runner你可能知道，Gitlab Runner可能有多种需求环境的Gitlab Runner，一般情况下不会将一个Runner安装所有应用环境，比如一般不会给Runner安装python、java、gradle、node.js、go等等一系列的，随着公司团队的不断壮大，技术栈的不断扩张，怎可能给一个GitlabRunner无限扩张环境？所以一般是针对前端、后台、或者说某种技术栈制作的一个GitlabRunner镜像来针对性的跑项目持续集成过程，

	**那么问题来了：**我们如何能让不同的Gitlab Runner环境和不同的项目进行一一对应呢？那就需要注册了啦！

	针对上面问题也许我们就会有如下解决方案，在含有JDK和Gradle或Maven构建工具的Gitlab Runner注册给A和B项目，C项目需要制作专门针对文档语法分析的镜像对其注册。


2. 注册过程中我该如何选择Gitlab Runner的executor？
	
基于docker的Gitlab Runner注册过程其实很简单,官方文档中是这样说的，

```html
docker exec -it gitlab-runner gitlab-runner register //执行注册命令
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com/ci      //输入注册Gitlab Ci地址，这个地址是你的Gitlab环境实例地址官方是gitlab.com,我们Gitlab实例的地址则是git.minixiao.com,这里需要注意
Please enter the gitlab-ci token for this runner
xxx    //输入项目注册Token
Please enter the gitlab-ci description for this runner
my-runner   //给此次注册的runner写说明
INFO[0034] fcf5c619 Registering runner... succeeded
Please enter the executor: shell, docker, docker-ssh, ssh?
docker   //选择runner的执行器
Please enter the Docker image (eg. ruby:2.1):
ruby:2.1  //如果选择了docker形式，就会让你用哪个docker image进行执行
INFO[0037] Runner registered successfully. Feel free to start it, but if it's   //提示注册成功，在项目页面Settings->Runners则能看见Specific Runners中有名为my-runner的Runner啦
running already the config should be automatically reloaded!
```

1. [官方executor选择说明](https://docs.gitlab.com/runner/executors/#selecting-the-executor)

| Executor                                          | Shell   | Docker | Docker-SSH | VirtualBox | Parallels | SSH  | Kubernetes |
|---------------------------------------------------|---------|--------|------------|------------|-----------|------|------------|
| Clean build environment for every build           | no      | ✓      | ✓          | ✓          | ✓         | no   | ✓          |
| Migrate runner machine                            | no      | ✓      | ✓          | partial    | partial   | no   | ✓          |
| Zero-configuration support for concurrent builds  | no (1)  | ✓      | ✓          | ✓          | ✓         | no   | ✓          |
| Complicated build environments                    | no (2)  | ✓      | ✓          | ✓ (3)      | ✓ (3)     | no   | ✓          |
| Debugging build problems                          | easy    | medium | medium     | hard       | hard      | easy | medium     |

我这里简单展示了他们的难易程度，此表来自于官方文档

2. 根据自身情况、项目情况和团队情况选择

看情况需要选择executor，我们在最初时选择的是docker，docker执行器环境会很干净，但是是无状态的，很多时候需要重复下载依赖jar包，所以后面我们选择了shell方式执行，脚本运行简单，执行简单，可控性还好。

3. 触发Gitlab Runner的先决条件是什么？
	
	1. 项目仓库根目录下中添加`.gitlab-ci.yml`文件

	2. 手动添加方式和GUI方式添加
		* 手动添加
			1. 创建`.gitlab-ci.yml`文件放项目根目录即可
			2. 输入CI过程的脚本文件
			3. push进Gitlab,自动执行脚本过程	
		* WEB GUI方式添加
			Gitlab项目下，找到`Set up CI`,意思和手动差不多，只是不用自己创建文件，直接输入脚本即可，好处在于此页面可以选择`template`,可以看到很多参考脚本。
	3. 添加CI脚本之前最好进行检验

		我们既然在写CI过程的脚本，肯定不想脚本出现语法错误，CI脚本语法可以查看[文档](http://docs.gitlab.com/ce/ci/yaml/README.html)，就算你熟悉了其语法，难免还是会出错，所以我们可以在写脚本时进行检查，在此页面[https://gitlab.com/ci/lint](https://gitlab.com/ci/lint)可进行语法检查。

更多内容给查看:[Gitlab Runner在Gradle构建项目中的应用](/2017/01/09/gitlab-runner-gradle/)





