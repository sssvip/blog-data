date: 2017/1/9
title: JAVA后端工作流推荐六--Gitlab Runner在Gradle构建的SpringBoot项目中的应用--实战篇
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
更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](/2017/01/10/workflow/)

可通过上一篇[Gitlab Runner对SpringBoot应用进行持续集成--理论篇](/2017/1/9/gitlab-runner/)了解Gitlab CI相关知识。

#### 此文阅读完效果:

1. 实现项目的自动化代码静态分析，单元测试，build等过程。
2. 熟悉Gitlab CI的Runner配置过程
3. 熟悉gradle中使用checkstyle

<!-- more -->

上面的理论知识你可以当作是经验之谈，就像实验一样，需要可重现才是真，下面接着上一篇教程进行实践，继续使用我们的helloword进行作为示例。

1. 添加Checkstyle服务，采用gradle插件方式进行
	我们先前已经[IDEA插件安装及使用](/2017/01/05/idea-plugins/)小节中在/helloworld/config/checkstyle/checkstyle.xml放置了checkstyle样式文件，我们现在来好好利用它。
	
	在项目根目录下`build.gradle`文件末尾添加如下代码，
	```html
	apply plugin: 'checkstyle'
	checkstyle {
    toolVersion = "7.3"
    checkstyleTest.enabled = false
	}
	/*防止warn 后继续构建，抛出异常后停止构建，达到提醒修改作用*/
	tasks.withType(Checkstyle).each {
	    checkstyleTask ->
	        checkstyleTask.doLast {
	            reports.all {
	                report ->
	                    def outputFile = report.destination
	                    if (outputFile.exists() && outputFile.text.contains("<error ")) {
	                        throw new GradleException("There were checkstyle warnings! For more info check $outputFile")
	                    }
	            }
	        }
	}
	```
	> 意思为启用gradle的checkstyle插件，具体知识可以查看gradle checkstyle文档，很简单的一页文档，2分钟看完的那种。

	添加后可以执行gradle checkstyleMain //此命令用于检测所有主要代码不包括测试代码。
	简单说明，gradle checkstyleMain 会去读取我们放在/helloworld/config/checkstyle/checkstyle.xml文件进行作为检验标准，这是它的默认路径，你可以更改路径，具体查看文档，这里不再赘述。 

	我们这里显示`BUILD SUCCESSFUL`是因为上次我们在演示插件安装时进行过checkstyle代码修正，等等在后面的环节我们会进行试错演示。

	![gradle-checkstyle](https://sssvip.github.io/img/gitlab-runner/gradle-checkstyle.png)

2. 添加`.gitlab-ci.yml`文件意味着申明进行持续集成
	
	在项目根目录添加`.gitlab-ci.yml`文件，并写简单脚本，语法可查看[文档](http://docs.gitlab.com/ce/ci/yaml/README.html)
	
	内容如下：

	![add-ci-yml](https://sssvip.github.io/img/gitlab-runner/add-ci-yml.png)

	其它语法不细说了，具体查语法文档，我们在script脚本(这个脚本中语法和你的Runner的executor是有关系的，后面Runner会选择shell作为executor,所以这里的语法是shell形式的，后面后具体看到我们在哪里选择shell)中的含义是gradle清除当前项目环境，然后检查main文件夹中的java代码的style,如果失败则会停止不再执行后续的build过程。

	build过程包含单元测试的过程，所以这是很好的检验项目的质量，但是前提是你要写单元测试:).
	
	我们当前状态下不注册Runner,将代码push上Gitlab,试试效果。

	![add-ci-yml-push.png](https://sssvip.github.io/img/gitlab-runner/add-ci-yml-push.png)
	
	![add-ci-yml-push-result.png](https://sssvip.github.io/img/gitlab-runner/add-ci-yml-push-result.png)
	
	如果说我们是自己搭建的gitlab服务的话，我们此步骤下是不会有Running,更不会有failed这些结果的，因为我们没有注册Runner(没有提前设置过shared的Runner)，为什么我提交到gitlab.com会出现构建失败的效果呢？你可以看到下图，他选用了gitlab.com提供的shared的Runner,`Running with gitlab-ci-multi-runner 1.9.0 (82714ae)`,他用的这个runner,仔细看下方命令是使用的ruby:2.1的docker进行执行的，他并没有gradle相关的环境，我们也没必要在这个CI脚本中写先安装jdk,再安装gradle等等，我们注册一个自己的包含gradle和jdk环境的runner即可。
	

3. 注册Runner

	> 如果你所在的Gitlab实例中有你shared的runner并且有你需要的运行环境可以跳过此步骤，目前公司项目中是存在gradle-jdk环境的公共runner的，所以此步骤可以跳过，如果被删除，需要联系管理员添加shared的runner，以便大家创建项目后无需再次注册此环境的runner。官方文档说明过，shared的runner只能由管理员添加，可以将shared的runner转为项目特有的，以及项目特有的runner不能转为shared等说明，具体可以查看官方文档。
	
	我们准备为项目添加包含上文提到的包含gradle,jdk1.8环境的为我们项目服务的runner:
		
	![add-runner](https://sssvip.github.io/img/gitlab-runner/add-runner.png)
	
	可以看到上图是存在Shared Runners的(如果存在Shared Runner,配置了`.gitlab-ci.yml`的项目会自动选择一个shared Runner进行执行CI过程)，我们需要添加针对我们项目服务的Specific Runners,上图中`How to setup a specific Runner for a new project`已经说明很详细，前面安装就不再赘述,下面的演示是基于安装好docker服务的计算机进行的,添加步骤如下图：
	
	![gitlab-runner-register](https://sssvip.github.io/img/gitlab-runner/gitlab-runner-register.png)
	
	简单解析：
	
	```html
	[root@iZ25vpysmgkZ ~]# clear //清空当前屏幕命令记录      
	[root@iZ25vpysmgkZ ~]# docker run -d --name gitlab-runner --restart always --privileged  -v /srv/gitlab-runner/config:/etc/gitlab-runner  startext/gitlab-runner-gradle  //启动startext/gitlab-runner-gradle镜像，-v 参数做给容器挂在主机目录， --name 取别名  -d 守护形式启动容器 具体的可以查docker文档，都是些比较常用的命令
	1be757d370125c24c3ddf7e858165564e45d377e3190a46f51dafed7413c0e4b
	[root@iZ25vpysmgkZ ~]# docker exec -it gitlab-runner gitlab-runner register
	Running in system-mode.  //exec 执行命令,-it 交互模式，整句意思就是在名为gitlab-runner 的容器执行gitlab-runner register命令                         
	                                                   
	Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/ci):
	https://gitlab.com/ci  //输入gitlab的地址，自己搭建的gitlab则输入自己的地址。
	Please enter the gitlab-ci token for this runner:
	KxCc7zzNFF4XXXXXXXXXXXX   //针对项目的token,鉴别项目还有身份
	Please enter the gitlab-ci description for this runner:
	[1be757d37012]: gradle  //描述，根据情况自己输入
	Please enter the gitlab-ci tags for this runner (comma separated):
	gradle   //tag标签
	Registering runner... succeeded                     runner=KxCc7zzN
	Please enter the executor: shell, ssh, virtualbox, docker+machine, docker-ssh+machine, docker, docker-ssh, parallels:
	shell   //选择executor，这里选择最简单易懂的shell,具体可以根据项目和自身情况选择
	Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
	[root@iZ25vpysmgkZ ~]# 

	```
	> 刷新gitlab页面，可以看到gitlab-runner已经添加	
	
	![add-runner](https://sssvip.github.io/img/gitlab-runner/add-runner-result.png)

4. 测试效果
	1. #### 命名不规范强行提交试错
		> 保证提交代码的风格符合checkstyle.xml约定，不通过则报错误，达到提醒修改的作用

		创建domain package,并创建User类，申明成员变量`Name`,此处命名不规范，IDEA明显有黄色警告，我们通过下方的插件进行检测也看出，`Member name`的`Name`必须符合正则表达式`小写字母开头的驼峰命名法。

		![checkstyle-user-name-test](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test.png)

		这里明显出错了，我们强行提交试试Gitlab CI效果,提交后看到在running：

		![checkstyle-user-name-test-running](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-running.png)
		
		![checkstyle-user-name-test-failed-again](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-failed-again.png)

		再次失败了，这里gitlab.com上的CI默认选择了上次的shared runner,这是不符合我们需求的，其实可以针对tag申明选择runner,这里我们简化先不说如何使用tag什么runner,我们直接关掉shared runner,使其默认使用我们注册的runner。
	
		![checkstyle-user-name-test-close-shared-runner](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-close-shared-runner.png)

		关掉后修改个小文件进行再次push(我此次修改的是一个代码注视,多加一个“.”,以便可以再次用git进行push,不然没有版本变化是不能进行push的)，我们查看下面的CI效果，我们CI脚本(`.gitlab-ci.yml`)写的是先checkstyle,然后在进行构建，这里我们预期是会在checkstyle这一步会失败，看看具体效果(第一次进行构建是会下载相关依赖，可能比较久，稍微等一等)

		![checkstyle-user-name-test-result](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-result.png)
		
		可以看到这里已经选用我们的gitlab runner进行执行CI过程，下载了众多依赖后符合我们预期的在checkstyle过程完美的失败了，我们详细看下具体内容：

		![checkstyle-user-name-test-result-2](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-result-2.png)

		![checkstyle-user-name-test-result-3](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-result-3.png)

		如果在安装gitlab的管理员配置了email服务的话(gitlab.com默认是有的，自己搭建的gitlab需要配置)，你每次的提交结果是可以收到CI结果邮件的，以便及时修正项目

		![checkstyle-user-name-test-result-email](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-result-email.png)

		![checkstyle-user-name-test-result-email](https://sssvip.github.io/img/gitlab-runner/checkstyle-user-name-test-update.png)

		我们将成员变量名进行修正，再次push上gitlab，进行测试效果：

		![checkstyle-pass-checkstyleMain](https://sssvip.github.io/img/gitlab-runner/checkstyle-pass-checkstyleMain.png)
		
		这里明显看到我们经过修正的代码是能通过CI过程中checkstyleMain的了，这就起到了很好的提交代码风格检测的作用。

		提示：此时项目后续的gradle build可能会失败，因为失败在gradle build包括了checkstyleTest过程，这个过程会失败，因为我们没有对Test代码进行代码风格修正，这里读者自行修复提交就行。

	2. 单元测试失败强行提交试错
		
		> 此举为了保证提交代码的单元测试能通过，不通过则报错误，达到提醒修改的作用
		
		![ci-test](https://sssvip.github.io/img/gitlab-runner/ci-test.png)
		
		这里我们进行单元测试演示，在实际项目中根据约定粒度，预定测试方法，进行测试即可，当然需要遵循单元测试规则，这里我们选用了junit进行掩饰，只作演示，我直接在HelloworldApplocationTests中进行演示，删掉初始代码，写入代码如上图，在IDEA中我们运行单元测试能看到效果，已经报错，我们希望的10，实际值是20，这里的assertEquals语法可以查询junit文档，相对来说是很简单的。

		这里我们已经知道错误了，模拟我们在不知情的情况下，或者明知单元测试通不过，还要强行提交代码情况，push至gitlab查看效果：

		这里我们也许还会出错,爆出类似这样`静态引包`的错误，如果项目允许我们可以跳过单元测试的checkstyle,不然我们需要修改checkstyle.xml进行允许此项要求或则单元测试写法上的修改。
		```html
		:checkstyleTest[ant:checkstyle] [WARN] /home/gitlab-runner/builds/16035ea7/0/sssvip/helloworld/src/test/java/com/minixiao/HellowordApplicationTests.java:5: Import statement for 'org.junit.Assert.assertEquals' is in the wrong order. Should be in the 'STATIC' group, expecting not assigned imports on this line. [CustomImportOrder]
		```
		我们在`build.gradle`中加一行代码，申明跳过checkstyleTest
		```html	
		checkstyle {
			toolVersion = "7.3"
			checkstyleTest.enabled = false //跳过测试代码的风格检验
		}
		```
		再次push查看我们的单元测试在CI过程的影响：

		![ci-test-failed](https://sssvip.github.io/img/gitlab-runner/ci-test-failed.png)

		可以看到这个单元测试失败了，此次CI过程也就意味着失败了，你就会再次收到失败的提醒邮件，所以及时修复问题。

		我们将断言改为正确的断言
		```html
		@Test
		public void numberTest() throws Exception {
			//这里仅作单元测试失败演示
			//断言10+10=10
			//assertEquals(10, 10 + 10, 0);
			assertEquals(20, 10 + 10, 0);
		}
		```
		![ci-test-correct](https://sssvip.github.io/img/gitlab-runner/ci-test-correct.png)

		再次提交查看CI结果：
		
		![ci-test-correct-result](https://sssvip.github.io/img/gitlab-runner/ci-test-correct-result.png)
		
		查看构建历史：

		![ci-test-correct-result-many](https://sssvip.github.io/img/gitlab-runner/ci-test-correct-result-many.png)

		每次构建收到的邮件:

		![ci-test-correct-result-many-email](https://sssvip.github.io/img/gitlab-runner/ci-test-correct-result-many-email.png)

		经过多次试错，我们修复后终于收到了pass的邮件(这里项目的CI脚本仅包括了主要代码的静态分析，build过程(包括了单元测试，项目打包过程等等)，你还可以根据需要，将build后的jar包进行启动，然后做自动发布测试环境什么的，这都是在脚本里面写的东西啦，做到个性化脚本操作。

5.Gitlab CI简单小结

整个CI过程最重要的两个关键是`.gitlab-ci.yml持续集成配置脚本`和`Gitlab Runner`,Gitlab CI在官方推荐的Runner这一块的持续集成主要通过两个对持续集成过程进行控制，写好一个`.gitlab-ci.yml`脚本文件，选好一个适合项目的Gitlab Runner executor都是非常重要的，这里可以多查看官方文档的说明，做到适合项目的力度就ok,不必追求过度复杂和自动化，根据项目按需选择即可。

原文地址: [https://blog.dxscx.com/2017/01/09/gitlab-runner-gradle/](https://blog.dxscx.com/2017/01/09/gitlab-runner-gradle/)