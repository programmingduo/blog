---
layout: post
title: "使用Sublime Text 3写Markdown"
description: "在Windows下配置sublime text 3用以编辑markdown文件。"
categories: [tools]
tags: [sublime,markdown]
redirect_from:
  - /2013/04/22/
---

* Karmdown table of content
{:toc .toc}

# 安装package controller
打开Sublime Text3，按下组合键Ctrl + `，出现控制台，输入:

~~~~~~~
import urllib.request,os;pf = 'Package Control.sublime-package';ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) );open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
~~~~~~~~~~~~~~~~

安装完成后打开Preferences就能看到,打开Package Control.

# 安装Markdown Preview和Markdown Editing
打开Package Control,输入install package，稍等一会就会出现很多插件，搜索你要的插件然后安装.
在插件安装面板输入markdown找到Markdown Preview并点击安装即可。

# 快捷键设置
Sublime Text支持自定义快捷键，markdown preview默认没有快捷键，我们可以自己为preview in browser设置快捷键。方法是在Preferences -> Key Bindings User打开的文件的中括号中添加以下代码(可在Key Bindings Default找到格式)：

~~~~~~~~~~~
{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"}  }   
~~~~~~~~~~~~~~~

"alt+m"可设置为自己喜欢的按键。 