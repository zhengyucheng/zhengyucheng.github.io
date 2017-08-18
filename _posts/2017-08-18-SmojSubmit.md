---
layout:     post
title:      "SmojSubmit"
subtitle:   "一键提交到SMOJ，无需多余操作"
date:       2017-08-18 09:58:00
author:     "Monad"
header-img: "img/post/SmojSubmit/Background.png"
mathjax:    false
tags:
    - Sublime
---

### 缘由
我们的学校自己开发了一个题库，我们都叫它SMOJ。但是由于它用的是教育网，所以硕慢无比，而且有时候还会抽风。  
所以写了代码，玄学调试过样例，准备提交的时候，又要改、加上文件读写，真是硕麻烦，就非常不爽。于是我就想，写一个程序来帮自己提交程序。

### 构想
但是如果要IDE外面运行这样的程序的话，还不如不用会。所以要做也要做Sublime的插件。  
然后插件又如何获取你要提交的题号呢？难道要检测`freopen`？我觉得不行，因为我们平时调试都是用`freopen("Temp.in", "r", stdin);`的，检测不到题号。那么除了这个以外，也没有什么可以作为判断了。  
那么既然没什么可以判断了，那就只能新增规则了。征求了我旁边的几位OIer之后，决定用`//1234.cpp`这个注释作为题号的标志。  
所以最终就得出了这么一个流程图：
![](/img/post/SmojSubmit/FlowChart.svg)

### 编写
在编写的过程中，主要有两个障碍，一个是Sublime插件的使用姿势，另一个是Python的`urllib`的使用。
#### Sublime plugin
Sublime Text 的插件我之前没有写过，Google了一下之后，就开始学习一下。
Sublime Text 的插件的class是用大驼峰式命名法，而且类名结尾必须是`Command`，否则Sublime不会识别。然后在这个类里面要定义一个`run`方法，Sublime调用这个类的入口就是`run`。
``` python
class SmojSubmitCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        pass
```
然后在Sublime的`Package`目录中，新建一个文件夹，名字就是你的插件的名字，然后把这个`.py`文件放入这个文件夹内。  
然后调用这个类的方式就是在Sublime的`Console`中，输入`view.run_command('class_name')`。然后这里的class_name和上面的类名不一样。这里的命名是上面的类名去掉Command之后，用下划线小写命名。例如`SmojSubmitCommand`就是`smoj_submit`。  
不过每次都打开`Console`来运行，很不方便。所以可以为它定义一个右键菜单项。首先在你的包文件夹内新建一个`Context.sublime-menu`文件，然后在里面输入：
``` json
[
    { "caption": "-" },  // 开始分隔符
    {
        "caption": "<显示名称>",
        "command": "<运行命令>",  // 和上面在Console中的class_name一样
        "args": {}
    },
    { "caption": "-", "id": "end" }  // 结束分隔符
]
```
然后保存，（最好）重启Sublime，你就会发现在右键菜单中多了一个选项。  
编写完一些环境设置之后，就要编写插件的主体部分了。  
首先就是找出缓冲区的题号标志符`//1234.cpp`。翻了一下Sublime API之后，发现搜索可以调用`find`。而且我还惊奇地发现，`find`居然支持正则表达式，这不是硕棒无比吗？直接来一个`// ?(\d{4,})\.cpp`正则就好了。
``` python
class SmojSubmitCommand(sublime_plugin.TextCommand):
    # ...
    def getProblemNum(self):
        chunk = self.view.find_all(r'// ?(\d{4})\.cpp', 0)
        if len(chunk) < 1:    # 没找到
            return None
        chunk = chunk[0]   # 取第一个的区间
        cpp_name = self.view.substr(sublime.Region(chunk.a, chunk.b))  # 取匹配到的字符串
        m = _cpp_re.search(cpp_name)
        cpp_num = m.group(1)   # 获取`//1234.cpp`中的题号
        return int(cpp_num)
```
然后就要去掉`freopen`的注释并且把里面的文件名改对。这个步骤也可以用“万能”的正则表达式完成。
``` python
class SmojSubmitCommand(sublime_plugin.TextCommand):
    def fillFreopen(self, content, problem):
        _fre_re = re.compile(r'freopen\("([^.])+\.(in|out)"( ?), "(r|w)", std(in|out)( ?)\);')
        _cm1_re = re.compile(r'/\*(\s*)((freopen(.*,.*,.*)\s*){1,2})\s*\*/')
        _cm2_re = re.compile(r'(\s*)//(\s*)(freopen\("([^.])+\.(in|out)"( ?), "(r|w)", std(in|out)( ?)\);)')
        result = content
        result = re.sub(_fre_re, r'freopen("%d.\2"\3, "\4", std\5\6);' % problem, result)
        result = re.sub(_cm1_re, r'\1\2'                                        , result)
        result = re.sub(_cm2_re, r'\1\2\3'                                      , result)
        return result
```
这样，我们就完成了提交之前的准备操作。  
提交的话，就比较少涉及到Sublime插件的编写规范问题了，但是要涉及网络通讯。

#### Python urllib
要想用Python `urllib`把代码submit上去，我们就要了解一下SMOJ的POST请求中有哪些成分。我们可以用Chrome自带的Debug功能查看。我先提交一次，从下面的图可以看到：
![](/img/post/SmojSubmit/PostPackage.png)
Chrome给SMOJ Post了3个数据，一个是题目的编号`pid`，一个是要提交的源文件类型`language`，另一个是你要提交的代码`code`。  
但是我们直接Post是不行的，因为我们还要登录。登录嘛，我们再用Chrome的debug功能抓一次包。
![](/img/post/SmojSubmit/LoginPackage.png)
这次的包有三个数据，`redirect_to`为空，我猜它应该是决定登录之后重定向到哪里；`username`为你的用户名；`password`就是密码的明文。  
我们Post了登录数据之后还要保留这个`cookie`，因为它在submit的时候还要用到。所以我选的是`urllib`中的`opener`。  
首先要生成一个`opener`对象：
``` python
cookie  = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener  = urllib.request.build_opener(handler)
```
然后`cookie`和`handler`就没什么用了，下面只需要用到`opener`。然后就要登录。  
登录可以用`r=urllib.request.Request(url, data, headers)`加上`opener.open(r)`来实现。这里的`data`的类型是`bytes`，但是自己拼接`data`有点麻烦，可以用`urllib.parse.urlencode(dict).encode()`把一个`dict`编码成一个`bytes`类型，而且无需担心url的转码问题。然后`headers`就直接传一个`dict`就好了。  
``` python
values = {'redirect_to':'', 'username':username, 'password':password}
r = urllib.request.Request(url=config.root_url+'/login', data=urllib.parse.urlencode(values).encode(), headers=headers)
response = opener.open(r)
```
登录后还要判断是否登录成功。因为我们这个题库如果不登陆的话，你只能访问login页面，只有登录了才能看里面的内容。如果登录成功，默认是跳转到首页。因为`Request`会自动处理重定向，所以这里可以检测当前url是否不为登录页面就行。  
登录完成后，`opener`里面就存储了用于验证身份的cookie，之后可以直接拿`opener`来用，也不用导出cookie，很方便。  
Post也类似，只要把`url`换成提交代码的url，再把`data`换成提交代码的data就行了。至于判断是否提交成功（我们题库有一些题是没权限提交的），则可以判断当前url是否为`/allmysubmits`即可。
``` python
values  = {'pid':str(problem), 'language':'.cpp', 'code':cpp}
r = urllib.request.Request(url=(post_url % problem), data=urllib.parse.urlencode(values).encode(), headers=headers)
response = self.opener.open(r)
```
然后这个一键提交的功能就愉快地写好了。

### 多线程
但是这样会有一个问题，因为我们的题库的网络很慢，如果用单线程（默认）的话，一提交就会卡，直到submit完成。  
所以应该要对网络操作新建一个线程。线程的话就和Sublime插件接口没有多大关联，直接用Python的写法就可以了。  
多线程执行的内容要写在一个`class`里面，这个`class`要继承自`threading.Thread`。如果要写`__init__`的话，记得在`__init__`的最后调用`threading.Thread`的`__init__`。调用这个`class`：
``` python
thread = ThreadingClass() # 这里的参数是传给__init__的
thread.start()
```
这样就完成了多线程的编写。  
同理，也可以对login进行多线程支持。

### 参考
简书 Rajnl：[Sublime Text 插件开发流程](http://www.jianshu.com/p/e2558ee1d503)  
[Sublime API 文档](https://feliving.github.io/Sublime-Text-3-Documentation/api_reference.html)  
我的[代码](https://github.com/YanWQ-monad/SmojSubmit)
