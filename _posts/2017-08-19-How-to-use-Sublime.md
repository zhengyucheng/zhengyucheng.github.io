---
layout:     post
title:      "Sublime Text"
subtitle:   "最“性感”的编辑器"
date:       2017-08-20 11:13:00
author:     "Monad"
header-img: "img/post/How-to-use-Sublime/Background.jpg"
mathjax:    false
catalog:    true
tags:
    - Sublime
---

### 介绍
Sublime Text 不仅是一款具有代码高亮、语法提示、自动完成且反应快速的编辑器，而且支持插件扩展，定制你专属的编辑器。它比Vim更好上手，比VS更轻便，比Notepad++、Dev-C++更漂亮，而且它跨平台并且支持 Windows XP，是你coding不二的选择。  
Sublime Text 3 比 Sublime Text 2 启动更快，而且基于Python 3，所以下文都是以 Sublime Text 3 为基础。

### 安装
首先，你可以从官方网站下载[Sublime Text](https://www.sublimetext.com/3)。  
然后，双击`Setup.exe`打开安装界面，一路按`Next`即可。其中有一个选项为`Add to explorer context menu`，如果选中它就会在文件的右键菜单中添加`Open with Sublime Text`选项，这个可以根据个人喜好选择。  

### 配置
一个好的编辑器拥有较多的配置选项，以符合个人的使用习惯。所以使用Sublime Text前要先进行配置。  
首先选择菜单中的`Preferences → Brower Packages...`，打开包文件夹，然后打开`User`文件夹。我觉得自带的C++编译脚本在Windows下会爆奇怪的错误，所以我就自己写了一个脚本。  
使用方法：在`User`中创建文件`Monad Cpp.sublime-build`，然后复制、粘贴以下内容并且保存。
``` json
{
	"shell_cmd": "g++ \"${file}\" -o \"${file_path}/${file_base_name}\" -Wall -O2 -g -std=c++14 -lwininet -static-libstdc++ -static-libgcc -lws2_32",
	"file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
	"working_dir": "${file_path}",
	"selector": "source.c++, source.cpp, source.cxx",
	"encoding": "cp936",

	"variants":
	[
		{
			"name": "Run",
			"shell_cmd": "echo Compiling ${file} =^> ${file_base_name}.exe && g++ \"${file}\" -o \"${file_path}/${file_base_name}\" -Wall -O2 -g -std=c++14 -lwininet -static-libstdc++ -static-libgcc -lws2_32 && echo Running ${file_base_name}.exe && \"${file_path}/${file_base_name}\""
		},
		{
			"name": "Only run",
			"shell_cmd": "\"${file_path}/${file_base_name}\""
		},
		{
			"name": "Run in Console (compile)",
			"shell_cmd": "echo Compiling ${file} =^> ${file_base_name}.exe && g++ \"${file}\" -o \"${file_path}/${file_base_name}\" -Wall -O2 -g -std=c++14 -lwininet -static-libstdc++ -static-libgcc -lws2_32 && echo Running ${file_base_name}.exe && start ConsolePauser.exe \"${file_path}/${file_base_name}.exe\""
		},
		{
			"name": "Run in Console (Run only)",
			"shell_cmd": "start ConsolePauser.exe \"${file_path}/${file_base_name}.exe\""
		}
	]
}
```
然后打开一个`cpp`文件，按下`Ctrl`+`Shift`+`B`，你应该会看到这个弹出框。因为我装了一个主题插件，所以样式看起来有一点不同，这没什么关系。
[![Build With](/img/post/How-to-use-Sublime/BuildWith.png)](/img/post/How-to-use-Sublime/BuildWith.png)
前两个`C++ Single File`是 Sublime 自带的编译脚本。后面五个是我的脚本的编译选项。各个选项作用如下：
* `[Default]` 只编译，不运行；
* `Run` 编译并运行。
* `Only run` 不编译，只运行；
* `Run in Console (compile)` 编译，并弹出控制台窗口运行。与上面的`Run`区别。
* `Run in Console (Run only)` 不编译，弹出控制台窗口运行。  

而且Sublime还可以用`Ctrl`+`B`直接运行上一次的命令。

#### 添加g++至 PATH
如果你现在不能正确地执行上面的编译选项，那么请检查你是否把g++所在目录添加至PATH。步骤如下：
1. 找出你g++.exe所在目录，并且复制：
[![g++ PATH](/img/post/How-to-use-Sublime/CxxPath.png)](/img/post/How-to-use-Sublime/CxxPath.png)
2. 右键`我的电脑`，依次选择`属性`，`环境变量`，然后双击`PATH`，在`变量值`中添加`;`和刚刚复制的目录。
[![Set PATH steps](/img/post/How-to-use-Sublime/PathSteps.png)](/img/post/How-to-use-Sublime/PathSteps.png)
3. 如果要用上面的`Run in Console (compile)`和`Run in Console (Run only)`，请把Dev-C++根目录下的`ConsolePauser.exe`复制到上述g++所在的目录中。
4. 重启Sublime Text。  

好了，现在你可以优雅地使用Sublime Text了。

### 插件
Sublime不仅原生功能强大，而且还支持插件，这让你可以定制属于你的编辑器。插件的安装方式主要有两种：
1. **直接安装**
Sublime可以很方便地安装插件，只要把下载的插件解压到`Packages`（可以用`Preferences → Brower Packages...`打开）中，Sublime就会自动加载并激活插件。

2. **通过Package Control安装**
如果还是嫌第一种安装方式太慢了，那么可以使用Package Control来安装。通过`Tools → Command Palette...`打开命令面板，然后选择`Install Package Control`，等待它安装完成即可。
当Package Control安装完成之后，你就可以通过命令面板选择`Package Control: Install Package`，Package Control就会搜索可用插件，并且显示出来，然后选择一个你想要安装的插件，按下回车就可以了。Package Control还有更多功能，在命令面板输入`Package Control`就可以浏览可用命令。

>如果你没有找到`Install Package Control`选项，那么你可能正在使用旧版的Sublime Text。这时，你需要通过另一种方式安装Package Control。
>按下`Ctrl`+`` ` ``打开Console（如果快捷键冲突，则可以在`View → Show Console`打开），然后把以下代码粘贴进去并回车：
>* **Sublime Text 2**
>``` python
>import urllib2,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
>```
>* **Sublime Text 3**
>``` python
>import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
>```

#### 插件推荐
**无插件，不神器**，下面就介绍一些常用的插件。[ 回家再补图 ]

##### OmniMarkupPreviewer
对于经常写Markdown的人来说，`OmniMarkupPreviewer`是一款必不可少的插件。它可以让你**实时**预览你的Markdown，而且它还支持MathJax。它的预览还可以是局域网内，也就是说，你可以用另一台设备预览你正在编辑的Markdown。当你编辑完成之后，它还可以把内容导出为HTML等格式。

##### SublimeGDB
如果你之前是用Dev-C++，那么你应该记得它的GUI调试功能。Sublime也不会落后，也有一款插件支持GUI调试，那就是SublimeGDB。它调试时提供的信息比Dev-C++的还要多。例如调用栈、变量查看（结构体展开）、GDB命令行交互、多线程调试等。*比Dev-C++不知道高到哪里去了*。
[![SublimeGDB](/img/post/How-to-use-Sublime/SublimeGDB.png)](/img/post/How-to-use-Sublime/SublimeGDB.png)

##### Boxy Theme
这个是一个Sublime的主题插件，可以让你的Sublime变得更漂亮（扁平化），效果就跟上面那个图差不多。而且还能让侧边栏的颜色和代码窗口的颜色相匹配，不至于逼死某些强迫症。它还有几种不同的配色给你选择，详情请移步至[Boxy Theme](https://packagecontrol.io/packages/Boxy%20Theme)

##### WakaTime
时间都到哪里去了？美好的一天都跑到哪里去了？WakaTime可以记录你在每一个项目，每一个文件，每一种语言所消耗的时间，并且精确到秒。你就可以精确且高效地管理你coding的时间，不必在一些其它的地方浪费时间。如果你想的话，你还可以拿到社交网站上装13。
[![WakaTime](/img/post/How-to-use-Sublime/WakaTime.jpg)](/img/post/How-to-use-Sublime/WakaTime.jpg)
你只要在[WakaTime](https://wakatime.com/)上注册一个帐号，它就会给你一个`key`，然后把这个`key`输入进Sublime的WakaTime插件中，它就会记录你coding的时间，并且发送到服务器进行统计。

##### SublimeTmpl
SublimeTmpl这款插件可以在你新建文件的时候添加模板内容，这样你以后就不用手动给C++打头文件、给HTML输入HTML头，不用给Python加入encoding注释等等。而且你还可以修改原有的配置或者新增你自己的配置。

##### 剩下一些可以根据个人喜好或需求安装：
* **ConvertToUTF8**
  支持 GBK, BIG5 等编码
* **WordCount**
  安装后在右下角显示字数
* **ClickableURLs**
  让文件中的URL可以点击
* **YUI Compressor**
  压缩JS和CSS文件

### 实用技巧
#### Ctrl+D
有时候遇到命名冲突，需要批量修改变量名时，一般都会一个一个改或者用一键替换的功能。但是一个一个改效率慢，一键替换会误伤。在Sublime中，你可以选择你想要修改的变量名中的一个，然后用`Ctrl`+`D`快速选择下面的变量，就可以批量修改文件名，非常方便。如果不想选择`printf`中的名字或者其它正常的名字，则可以用`Ctrl`+`K`跳过，如果选过头了，还可以用`Ctrl`+`U`回退。
[![Ctrl+D](/img/post/How-to-use-Sublime/CtrlD.gif)](/img/post/How-to-use-Sublime/CtrlD.gif)
