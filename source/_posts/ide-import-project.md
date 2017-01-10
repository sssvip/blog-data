date: 2017/01/04
title: JAVA后端工作流推荐二--Gitlab创建项目并导入IDE
tags: 
- gitlab
- idea
- import
---
更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](/2017/01/09/workflow/)

#### 此文阅读完效果:
1. Gitlab的基本使用
2. IDEA的基本使用
3. Git的基本使用

<!-- more -->

### 创建Gitlab项目仓库
1. 创建帐号并登录
> 此次演示未在git.minixiao.com上创建帐号和而是在gitlab.com上创建helloworld项目进行演示，道理都一样。
	创建账户登录不用过多说明，内部用git.minixiao.com帐号注册通道已经关闭，注册需联系管理员。
2. 创建helloworld项目

![ide-import-prepare](https://sssvip.github.io/img/ide-import-project/new-project-prepare.png)

登录后点击Projects后能看见你当前的所有项目，点击**New Project**创建项目
![new-project](https://sssvip.github.io/img/ide-import-project/new-project.png)

1. **输入Project name 项目名称**
2. **输入Project desciption，虽然是可选的，还是比较重要，让人第一眼了解你的项目。**
3. **根据项目需要选择项目Visibility Level项目可见等级，helloworld演示项目
，这里直接公开即可。**
4. **点击Create project不出意外你可以看到下图的样子，恭喜你管理仓库已经建好了。**
![new-project-created](https://sssvip.github.io/img/ide-import-project/new-project-created.png)

### 从远端仓库Clone克隆至本地

1. [Git安装](https://git-scm.com/downloads),选择您对应系统进行下载安装
	
	> 安装没有什么难的，值得一提注意添加path变量:)

2. 克隆至本地
![clone-to-local](https://sssvip.github.io/img/ide-import-project/clone-to-local.png)

**命令解析:**

```html
Administrator@tangw-A-3 MINGW64 ~
$ git --version  //查看当前git版本
git version 2.10.1.windows.1

Administrator@tangw-A-3 MINGW64 ~
$ cd d:  //进入d盘

Administrator@tangw-A-3 MINGW64 /d
$ cd Demo/    //进如D盘下Demo文件夹，此次演示用的

Administrator@tangw-A-3 MINGW64 /d/Demo
$ git clone https://gitlab.com/sssvip/helloworld.git   //进行远端clone至本地，克隆地址在helloworld项目下project页面能看见，此次可能采用https方式，根据自己喜好可选择SSH
Cloning into 'helloworld'...
warning: You appear to have cloned an empty repository.  //提示克隆了一个空仓库

Administrator@tangw-A-3 MINGW64 /d/Demo
$ cd helloworld/  //进入到此次克隆项目的主目录

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master) //进入后能看见(master)标识,进入了版本管理状态，并且在master分支上
$

```

### 添加[上一步](/2017/01/03/springboot/)构建的项目初始文件至仓库，进行版本管理
将上一步构建好的项目初始文件移动至helloworld文件夹，最终效果如下：
![local-init-project](https://sssvip.github.io/img/ide-import-project/local-init-project.png)

**命令解析：**
```html
Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ ls   //查看当前目录下文件
build.gradle  gradle/  gradlew*  gradlew.bat  src/    //可以看见上一步构建的项目初始文件已经移动至当前helloworld本地仓库

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$
```

### 从本地仓库push推至远端仓库
将刚刚添加的项目文件存储到本地仓库并push推到远端gitlab仓库
![push-to-remote](https://sssvip.github.io/img/ide-import-project/push-to-remote.png)

**命令解析：**
```html
Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ git status //查看本地仓库状态--这里主要想看下本地仓库文件变化情况
On branch master

Initial commit

Untracked files:  //这里提示有未跟踪的文件
  (use "git add <file>..." to include in what will be committed)

        .gitignore
        build.gradle
        gradle/
        gradlew
        gradlew.bat
        src/

nothing added to commit but untracked files present (use "git add" to track)

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ git add -A  //提交当前变化的所有文件，git命令可上官方文档查一查
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in build.gradle.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in gradle/wrapper/gradle-wrapper.properties.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in gradlew.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in src/main/java/com/minixiao/HellowordApplication.java.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in src/test/java/com/minixiao/HellowordApplicationTests.java.
The file will have its original line endings in your working directory.

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ git commit -m "feat: init helloworld project file" //提交刚add的文件到本地仓库，-m后面我加了注视初始化项目文件
[master (root-commit) 5a81df6] feat: init helloworld project file
 9 files changed, 344 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 build.gradle
 create mode 100644 gradle/wrapper/gradle-wrapper.jar
 create mode 100644 gradle/wrapper/gradle-wrapper.properties
 create mode 100644 gradlew
 create mode 100644 gradlew.bat
 create mode 100644 src/main/java/com/minixiao/HellowordApplication.java
 create mode 100644 src/main/resources/application.properties
 create mode 100644 src/test/java/com/minixiao/HellowordApplicationTests.java

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ git push origin //push本地仓库文件到远端gitlab
Username for 'https://gitlab.com': sssvip   //提示输入账户，你刚注册或登录的那个账户，当然这里会叫你输入密码
Counting objects: 23, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (16/16), done.
Writing objects: 100% (23/23), 51.73 KiB | 0 bytes/s, done.
Total 23 (delta 0), reused 0 (delta 0)  //23个文件成功
To https://gitlab.com/sssvip/helloworld.git
 * [new branch]      master -> master  //本地master到远端master

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$
```
在远端仓库你能看见如下效果，项目文件就推送到远端啦！！！
![push-to-remote-after](https://sssvip.github.io/img/ide-import-project/push-to-remote-after.png)

### 导入IDE(IntelliJ IDEA)
1. 打开IDEA软件导入项目，点击**Import Project**进行项目导入
> 项目中用IDEA Community,用Eclipse道理一样，很多东西都是共通的。

![idea-import-project](https://sssvip.github.io/img/ide-import-project/idea-import-project.png)

选择刚刚的项目路径`D:\Demo\helloworld`进行导入

![idea-import-project-select](https://sssvip.github.io/img/ide-import-project/idea-import-project-select.png)

选择Import Project from external model 和Gradle项，这选项是默认的，因为IDEA已经识别到了

![idea-import-project-from-external](https://sssvip.github.io/img/ide-import-project/idea-import-project-from-external.png)

勾选Use auto-import，然后finish

![idea-import-project-auto-import](https://sssvip.github.io/img/ide-import-project/idea-import-project-auto-import.png)

首次使用SpringBoot或则首次使用新版本的SpringBoot会下载相应的以来jar包，稍微等等就好。

![idea-import-project-downloading](https://sssvip.github.io/img/ide-import-project/idea-import-project-downloading.png)

然后你看到了这个，恭喜你，项目已经导入成功，可以开始搞事情了。。。

![idea-import-project-helloworld](https://sssvip.github.io/img/ide-import-project/idea-import-project-helloworld.png)

**提示：** 可能存在意外的是注意看**Event Log**，如果出现类似情况(也许你没有)，将版本控制注册，后面就可以很好的使用Gui的Git,一般情况下可以不用敲git命令了，不过还是建议大家多使用git命令。
![idea-import-project-helloworld-git](https://sssvip.github.io/img/ide-import-project/idea-import-project-helloworld-git.png)

继续查看下一篇: [JAVA后端工作流推荐三--IDEA插件安装及使用](/2017/01/05/idea-plugins/)

原文地址: [https://blog.dxscx.com/2017/01/04/ide-import-project/](https://blog.dxscx.com/2017/01/04/ide-import-project/)