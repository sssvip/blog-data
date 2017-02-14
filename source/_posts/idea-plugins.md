date: 2017/01/05
title: JAVA后端工作流推荐三--IDEA安装Checkstyle、FindBugs、Markdown等插件及其使用
tags: 
- markdown
- idea
- Checkstyle
- FindBugs
- springboot
- team
- codestyle
---

更多JAVA后端工作流推荐内容参见:[JAVA后端工作流推荐系列---目录导航](/2017/01/10/workflow/)

#### 此文阅读完效果:

1. 基于在IDEA中安装Markdown、CheckStyle、FindBugs等插件
2. 基于在IDEA中书写Markdown文档、基于CheckStyle代码静态分析、基于FindBugs查找项目中潜在Bug
3. 最重要的是通过这种方式提升团队开发的写作效率，统一代码风格，以及通过Findbugs找自己code潜在问题。

<!-- more -->

## 插件安装
打开IDEA后，点击FILE->Settings->Plugins 可见到此图效果，安装下面插件
![plugins-enter](https://sssvip.github.io/static/img/idea-plugins/plugins-enter.png)

### CheckStyle插件安装
![plugins-checkstyle](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle.png)
### Markdown插件安装
> Markdown Navigator、Markdown support两组插件，当然您可以选择其它您自己喜欢的。

### FindBugs插件安装
> FindBugs插件有很多，推荐安装名为“FindBugs-IDEA”的这个插件，当然你看安装量显示就知道挺不错，这个插件目前的安装量有50W+。

![plugins-findbugs](https://sssvip.github.io/static/img/idea-plugins/plugins-findbugs.png)
Markdown插件安装，FindBugs插件安装方式和checkstyle安装截图一样，这里就不再截图。

## 插件配置使用
### 添加CheckStyle的xml配置文件及使用
安装完插件你应该会看到重启IDEA提示，没有的话就手动关闭重启一次，插件就加载啦。
再次进入Settings您就能看到多了一个Other Setting,我们刚安装的插件，这里没有显示markdown的插件，因为这个插件几乎不需要什么设置。

![plugins-add-checkstylexml](https://sssvip.github.io/static/img/idea-plugins/plugins-add-checkstylexml.png)

插件下载后Checkstyle-IDEA默认提供了sun_checks.xml,这里我们使用自己根据谷歌java code checkstyle 配置修改的符合自己项目的checkstyle.xml,这个checktyel配置文件路径放置如下:
`/helloworld/config/checkstyle/checkstyle.xml`,放在这有好处，后面会在gradle构建检测中用到，当然你也可以防止其它地方，根据自己需要来就行，这里我推荐这种方式而已。

![plugins-add-checkstylexml-2](https://sssvip.github.io/static/img/idea-plugins/plugins-add-checkstylexml-2.png)
![plugins-add-checkstylexml-3](https://sssvip.github.io/static/img/idea-plugins/plugins-add-checkstylexml-3.png)

配置基本好了，我们来使用试试，在底部导航栏您能看到CheckStyle和Findbugs插件的快捷方式啦

![plugins-usage-checkstyle](https://sssvip.github.io/static/img/idea-plugins/plugins-usage-checkstyle.png)

简单说明：在Rules可以选择代码风格检测规则，这就和上面我们添加的checkstyle.xml有关，这里我们选择刚刚我们命名的checkstyle规则，根据项目需要，可以引入和命名不同的规则，以便区分。这里就仅有默认提供的sun_checkstyle和我们自己加的checkstyle两套规则。

左边的导航快捷键包含了Check Project项目代码分析,Check Module检测模块代码，Check All Modified Files检测修改过的文件等等快捷方式，根据你自己的需求选择检测即可。
现在我们选择`Check Project`检测项目试试效果：

![plugins-usage-checkstyle-checkproject](https://sssvip.github.io/static/img/idea-plugins/plugins-usage-checkstyle-checkproject.png)

![plugins-usage-checkstyle-checkproject-preview](https://sssvip.github.io/static/img/idea-plugins/plugins-usage-checkstyle-checkproject-preview.png)


不同的规则会有不同的检测结果，我们的checkstyle.xml中约定public方法名缩进2字符，方法体内容同条件下再次缩进2个字符等等格式规则。我们可以手动的去修改每一行，每一句代码的缩进格式难道我们真的要一个一个去操作代码缩进？还要去记住这一大堆的规则吗？真的不用，下面就应该我们的Formatter上场啦,Formatter帮我们搞定这些复杂而又难记的格式问题，我们只需要安静的写代码就够啦。
### 基于CheckStyle的Formatter导入
> Formatter是干嘛的？Formatter是简单翻译是格式化工具对吧？格式化是不是对于工具来说需要知道格式的样式，需要格式成什么效果，细心的你可能已经发现，这就是在修改我们Java Code Style,这就是Formatter的格式化依据，我们这里是针对JAVA代码所说的情况，其他语言，其他文本格式同理。你也可以看见默认提供了Project和Default两个Code Style Schemes，但通过默认提供的Default提供样式格式化是不能通过我们checkstyle.xml样式检测的，所以我们就在想如何更快的让其自动的格式化成我们checkstyle.xml样式的格式，这就用到了“反向生成”概念，IDEA提供了通过CheckStyle Configuration导入Code Style，就是下图`7`步骤所示，简单不粗暴。

![plugins-checkstyle-formatter-import](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle-formatter-import.png)

![plugins-checkstyle-formatter-import-2](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle-formatter-import-2.png)

选择后默认名字是`Default(1)`,这里方便管理Save As将名字命名为你方便记忆和管理的名字，我这里为方便管理，更名为checkstyle（意为这个样式来源于checkstyle）,然后删除`Default(1)`即可

![plugins-checkstyle-formatter-import-3](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle-formatter-import-3.png)

![plugins-checkstyle-formatter-import-4](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle-formatter-import-4.png)

选择我们刚加入的checkstyle样式，Apply然后OK，我们来试一试

我在HelloworldApplication.java文件中按了`Ctrl+Alt+L`默认提供的代码格式化快捷键(如果你更改过快捷键请按自己的)，代码被快速格式化，然后我添加上了代码注释（这里的注释没有太多业务逻辑，代码注释用的JavaDoc插件生成），经过快捷键格式化你可以看见编辑区的黄色警告已经消失，还可以点击鼠标右键，选择Check Current File然后下方的CheckStyle Scan区域会显示结果，很明显Checkstyle提示`Checkstyle found no problems in the file(s)`

![plugins-checkstyle-formatter-import-5](https://sssvip.github.io/static/img/idea-plugins/plugins-checkstyle-formatter-import-5.png)

到这里我们的checkstyle基础使用已经结束，FindBugs方式是差不多的，左部导航栏，中部显示结果什么的，这个对于聪明的你没有太大问题，需要提示的是，多用findbugs检测自己的代码，根据提示进行必要的修复，很多建议是很不错的，能帮助自己发现很多没注意的code细节，不断完善自己的研发质量，一起加油。

![plugins-usage-findbugs](https://sssvip.github.io/static/img/idea-plugins/plugins-usage-findbugs.png)



### Markdown的使用
> 到目前位置我们的项目都还没添加项目README.md,按照国际惯例我们添加README.md至项目根目录，然后开始我们的项目README

Markdown编辑语法和使用不再赘述，推荐查看上次会议分享的[http://markdown.dxscx.com](http://markdown.dxscx.com)。

![plugins-markdown-preview](https://sssvip.github.io/static/img/idea-plugins/plugins-markdown-preview.png)
这里能看见我们在编辑的时候可以同时预览，这也就是我们安装插件的好处，同时可以看到有视图切换等等按钮，这里简单介绍Markdown的使用，后面会结合在同项目下直接编辑WIKI,同一IDEA下搞定工作流，免去切软件切屏等等麻烦。

此篇文章不仅仅让你学会了IDEA Checkstyle插件安装和使用，这能推广到IDEA其它插件的安装和使用，都是大同小异的，更多的东西一起去探索。

继续查看下一篇: [JAVA后端工作流推荐四--Gitlab中Wiki文档同项目空间管理](/2017/01/05/wiki-manager/)

原文地址: [https://blog.dxscx.com/2017/01/05/idea-plugins/](https://blog.dxscx.com/2017/01/05/idea-plugins/)