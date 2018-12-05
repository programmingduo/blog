---
layout: post
title: "windows环境下安装ruby和jekyll"
description: ""
categories: [environment]
tags: [ruby, jekyll, windows, devkit]
redirect_from:
  - /2013/04/22/
---

* Karmdown table of content
{:toc .toc}

# 安装 Ruby

Jekyll是一款基于Ruby的插件，安装Ruby是必须的. 
1. 下载，传送阵：<http://rubyinstaller.org/downloads/>
2. 点击版本并下载，（千万不要下载2.5.x的版本，我第一次下了最新版，然后一路安装最后发现版本不匹配，全部删了重新安装的）。这里我下载的是：“Ruby 2.3.3 (x64)”  

### 点击进行安装，此时需要注意两点： 
* 安装目录不允许包含空格 
* 选中“Add Ruby executables to your PATH”这样将自动完成环境变量的配置。

### 完成后进入“CMD”输入“ruby -v”如显示版本则代表安装成功。

# 安装 DevKit

DevKit 是一个在 Windows 上帮助简化安装及使用 Ruby C/C++ 扩展如 RDiscount 和 RedCloth 的工具箱。 
更多详细的安装指南请查看Ruby的 wiki（<https://github.com/oneclick/rubyinstaller/wiki/Development-Kit#installation-instructions>） 页面 阅读。

前往 <http://rubyinstaller.org/downloads/>

下载与 Ruby 版本相对应的 DevKit 安装包。 例如：“DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe” 

运行文件选择解压目录，如“C:\DevKit”

解压完成后，通过初始化来创建 config.yml 文件。在命令行窗口内，输入下列命令：

~~~~~~~~~~~~~
cd "C:\DevKit"
ruby dk.rb init
notepad config.yml
~~~~~~~~~~~~~~~~~

此时已经使用记事本打开所创建的”config.yml”文件，于末尾添加新的一行: 

~~~~~~~~
- C:/Ruby23-x64
~~~~~~~~~~~~~~

这里的目录为你的Ruby的安装目录，保存文件并退出。如果已经有这一行则无需添加。

回到命令行窗口内进行安装。

~~~~~~~~~~
ruby dk.rb install
~~~~~~~~~~~~~

# 安装 Jekyll
首先确保 gem 已经正确安装

命令输入
~~~~~~~
gem -v
~~~~~~~~~~~~~
输出
~~~~~~~~~
2.2.2
~~~~~~~~~~~~~~~

如果未安装则自行安装gem。

安装 Jekyll

//命令行执行
gem install jekyll

# 启动jekyll
按照官方的 Jekyll 快速开始手册 的步骤， 一个新的 Jekyll 博客可以被建立并在localhost:4000浏览。

~~~~~~~
jekyll new myblog
cd myblog
jekyll serve
~~~~~~~~~~~~~~~~~~

# 遇到问题

### 故障诊断：

~~~~~~~~~~
C:/Ruby23-x64/lib/ruby/2.3.0/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- bundler (LoadError)
~~~~~~~~~~~

解决方案：

~~~~~~~~~~~~
gem install bundler
~~~~~~~~~~~~~~~~~

### 故障诊断：

~~~~~~~~~~~
C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.16.1/lib/bundler/resolver.rb:289:in `block in verify_gemfile_dependencies_are_found!': Could not find gem 'minima (~> 2.0) x64-mingw32' in any of the gem sources listed in your Gemfile. (Bundler::GemNotFound)
~~~~~~~~~~~~~~~~~~~~

解决方案：同上

~~~~~~~
gem install minima
~~~~~~~~~~~~~

### 写在最后
另，这种问题有时候会出现很多，install了一个之后很可能又会出现另一种。一样的道理都可以使用以上方法解决。但是在本人安装的过程中出现了一下情况：

~~~~~~~~~~~~
C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.16.1/lib/bundler/resolver.rb:55:in `rescue in start': Bundler could not find compatible versions for gem "jekyll-redirect-from": (Bundler::VersionConflict)
~~~~~~~~~~~~~~~~~~

在使用命令 gem install jekyll-redirect-from之后同样出现了以上情况。最终一通百度之后找到了一个方法：
在你的blog根目录里使用命令：

~~~~~~~~~~~
bundle install
~~~~~~~~~~~~~~~~~~~

应该来说这一命令里面一次性install了相关插件
