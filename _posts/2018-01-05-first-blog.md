---
layout: post
title: "使用Github Pages建独立博客并链接域名"
description: "第一篇博客"
categories: [environment]
tags: [github page, jekyll]
redirect_from:
  - /2013/04/22/
---
这个是第一次使用jekyll写博客，使用的语言据说叫做markdown。有空我得好好学习一下这门语言。

为了防止自己忘记，赶紧写一下搭建blog中的各种问题吧。

**先写对使用者的要求：**

1. 网速良好。不良好也没关系，你自己能忍就行。
2. 有github账户以及Git环境。没有也行，赶紧搞一下。

参考博客：<http://beiyuu.com/github-pages>

上面的描述十分详尽了。此处复制并稍加编辑。

* Karmdown table of content
{:toc .toc}

# 购买、绑定独立域名

### 购买独立域名
域名的购买不用多讲，注册、选域名、支付，有网购经验的都毫无压力，优惠码也遍地皆是。购买地址：<https://www.godaddy.com/>

流传Godaddy的域名解析服务器被墙掉，导致域名无法访问，后来这个事情在BeiYuu也发生了，不得已需要把域名解析服务迁移到国内比较稳定的服务商处，这个迁移对于域名来说没有什么风险，最终的控制权还是在Godaddy那里，你随时都可以改回去。

### 更改解析ip
之后要使用DNSPod更改域名解析的ip地址。（狗爹会给你一个原始的ip地址给你玩玩，但是我们要把它改成我们的github page不是。DNSPod就是用来干这个的。）DNSPod网址：<https://www.dnspod.cn>。更改ip过程参考DNSPod的官方指南：<https://www.dnspod.cn/Support>。

其中，在DNSPod自己的域名下添加一条A记录，地址就是Github Pages的服务IP地址：151.101.229.147(由于这个地址可能会变，所以推荐你自己ping一下github page的主机ip。cmd命令：ping github.io）
在域名注册商处修改DNS服务:去Godaddy修改Nameservers为这两个地址：f1g1ns1.dnspod.net和f1g1ns2.dnspod.net。自己登陆账号慢慢找吧，相信你，程序员。

### 等待域名解析生效

# 配置和使用Github
### 检查SSH keys的设置
首先我们需要检查你电脑上现有的ssh key：

~~~~~~
$ cd ~/.ssh
~~~~~~~~~~~

如果显示“No such file or directory”，跳到第三步，否则继续。

### 备份和移除原来的ssh key设置：
因为已经存在key文件，所以需要备份旧的数据并删除：

~~~~~~~~~~~
$ ls
config	id_rsa	id_rsa.pub	known_hosts
$ mkdir key_backup
$ cp id_rsa* key_backup
$ rm id_rsa*
~~~~~~~~~~~~~~~~~

### 生成新的SSH Key：
输入下面的代码，就可以生成新的key文件，我们只需要默认设置就好，所以当需要输入文件名的时候，回车就好。

~~~~~~~~~
$ ssh-keygen -t rsa -C "邮件地址@youremail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):<回车就好>
~~~~~~~~~~~~~~

然后系统会要你输入加密串（Passphrase）：

~~~~~~~~~~
Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
~~~~~~~~~~~~~~

最后看到这样的界面，就成功设置ssh key了： ssh key success

### 添加SSH Key到GitHub：
在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。

用文本编辑工具打开id_rsa.pub文件，如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。

在GitHub的主页上点击设置按钮： github account setting

选择SSH Keys项，把复制的内容粘贴进去，然后点击Add Key按钮即可： set ssh keys

PS：如果需要配置多个GitHub账号，可以参看这个多个github帐号的SSH key切换，不过需要提醒一下的是，如果你只是通过这篇文章中所述配置了Host，那么你多个账号下面的提交用户会是一个人，所以需要通过命令git config --global --unset user.email删除用户账户设置，在每一个repo下面使用git config --local user.email '你的github邮箱@mail.com' 命令单独设置用户账户信息

### 测试一下

可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：

~~~~~~
$ ssh -T git@github.com
~~~~~~~~~~~~

如果是下面的反应：

~~~~~~
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
~~~~~~~~~~~~~~~~~~

不要紧张，输入yes就好，然后会看到：

~~~~~~~
Hi <em>username</em>! You've successfully authenticated, but GitHub does not provide shell access.
~~~~~~~~~~~~~~~

### 设置你的账号信息（在你的工作路径使用git命令时再做）
现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。
Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。

~~~~~~~~~~~~
$ git config --global user.name "你的名字"
$ git config --global user.email "your_email@youremail.com"
~~~~~~~~~~~~~~~~
好，到现在，我们就完成了本地到github的链接了。

# 使用GitHub Pages建立博客
与GitHub建立好链接之后，就可以方便的使用它提供的Pages服务，GitHub Pages分两种，一种是你的GitHub用户名建立的username.github.io这样的用户&组织页（站），另一种是依附项目的pages。

这里主要讲User & Organization Pages。

想建立个人博客是用的第一种，形如beiyuu.github.io这样的可访问的站，每个用户名下面只能建立一个。我们在自己github账户里面新建一个repositoriy。创建之后点击settings进入项目管理，把github pages点绿了。

创建好username.github.io项目之后，提交一个index.html文件，然后push到GitHub的master分支（也就是普通意义上的主干）。第一次页面生效需要一些时间，大概10分钟左右。（也可以直接download一个漂亮的jekyll模板。模板网址：<http://jekyllthemes.org/>我就是在这个里面找的。）（提示：这个过程中需要git的add，commit，push命令）
生效之后，访问username.github.io就可以看到你上传的页面了

关于第二种项目pages，简单提一下，他和用户pages使用的后台程序是同一套，只不过它的目的是项目的帮助文档等跟项目绑定的内容，所以需要在项目的gh-pages分支上去提交相应的文件，GitHub会自动帮你生成项目pages。具体的使用帮助可以参考Github Pages的官方文档：

# 绑定域名
我们在第一部分就提到了在DNS部分的设置，再来看在GitHub的配置，要想让username.github.io能通过你自己的域名来访问，需要在项目的根目录下新建一个名为CNAME的文件（一定要大写而且不要有后缀），文件内容形如：
~~~~~
beiyuu.com
~~~~~~~~~~~~

（也可以在我们repository里面的setting里面，在github page里面的custom domain里填写自己的买的域名。）

****
到这一步，应该来讲，无论你访问   你的github用户名.github.io/你的repository名  还是  你买的域名，都可以访问到你的博客了。

奥对了，这里还有一个问题。就是 你的github用户名.github.io/你的repository名  可能ping不通。因为你的网址可能不安全，而ICMP请求在某个地方被和谐了。无所谓的，不要担心，不影响你的使用。