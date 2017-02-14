date: 2017/01/06
title: JAVA后端工作流推荐四--Gitlab中Wiki文档同项目空间管理
tags: 
- markdown
- wiki
- gitlab
- terminal
---
更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](/2017/01/10/workflow/)

#### 此文阅读完效果:

1. Wiki基于项目同目录管理
2. 文本编辑效率提升

<!-- more -->

### Wiki目录初始化

初次访问Wiki目录是没有任何内容的,点击Wiki tab后我我们能看到创建Home页，Home页见名思意就知道这是主页，一般用来做项目Wiki说明，WIFI文档导航等等。

![wiki-init](https://sssvip.github.io/static/img/wiki-manager/wiki-init.png)

我们一般选择Markdown格式书写，也可个根据自己情况选择格式，这个会根据你选择的格式生成不同格式文件后缀文件。当然你可以看见可以添加附件的，Gitlab Wiki可以上传附件，其会根据文件hash产生唯一地址，不会担心重复上传什么的。做完初始填写后点击`Create page`,填错也不怕，后面可以修改。

![wiki-init-clone](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone.png)

### Wiki克隆至本地

Wiki页有右边"<<"按钮，点击后您可以看到`Clone repository`点击后你可以看到上面的页面，提示已经教会你如何进行管理它，clone后进行本地管理就行，我们这里要做的是同目录下用IDEA去管理，去编辑Wiki。

![wiki-init-clone-2](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone-2.png)

```
Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ git clone git@gitlab.com:sssvip/helloworld.wiki.git //克隆仓库到helloworld目录
Cloning into 'helloworld.wiki'...
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 7 (delta 1), reused 0 (delta 0)
Receiving objects: 100% (7/7), done.
Resolving deltas: 100% (1/1), done.

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld (master)
$ cd helloworld.wiki //进入Wiki仓库master分支

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld/helloworld.wiki (master)
$ ls //显示当前目录下文件，这就是我们刚创建的home page,由于选择的markdown格式，所以文件格式后缀是md
home.md

Administrator@tangw-A-3 MINGW64 /d/Demo/helloworld/helloworld.wiki (master)
$
```
![wiki-init-clone-3](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone-3.png)

### 添加Wiki并push至Gitlab的Wiki

打开IDEA你就能看见，这里已经有我们的Wiki文档了，我们创建一个`Test.md`,为方便直接复制home.md内容

![wiki-init-clone-4](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone-4.png)

![wiki-init-clone-push](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone-push.png)

Terminal中命令不再解释，为简单的版本管理流程，这里截图想说明的是，我们不但可以用IDEA提供GUI方式去版本控制，也可以用IDEA下方的Terminal进行用git命令方式进行版本控制，不用我们再开cmd窗口，这也是“一站式管理”的又一体现。

![wiki-init-clone-push-preview](https://sssvip.github.io/static/img/wiki-manager/wiki-init-clone-push-preview.png)

最终你的页面已经push进gitlab的Wiki啦，流程大致是这样，在项目中写好Wiki是一大挑战，需要针对项目进行结构化的管理。

继续查看下一篇: [JAVA后端工作流推荐五--Gitlab Runner对SpringBoot应用进行持续集成--理论篇](/2017/01/09/gitlab-runner/)

原文地址: [https://blog.dxscx.com/2017/01/06/wiki-manager/](https://blog.dxscx.com/2017/01/06/wiki-manager/)