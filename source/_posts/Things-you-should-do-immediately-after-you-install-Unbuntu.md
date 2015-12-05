title: Things you should do immediately after you install Unbuntu
date: 2015-12-05 10:01:55
tags: [Ubuntu]
---
前天晚上为了在我那破旧的Y450上装Windows 10和Ubuntu 14.04的双系统，折腾了两天。最后成功把Windows 10系统给装没了。总结原因主要是有两点，一是没有搞清楚如何在双硬盘（我的主硬盘是128G的SSD，光驱位置放的是原来320G的HDD）条件下给两个盘装两个系统，始终纠结于为何SSD上的启动器找不到HDD上的Ubuntu；二是没有搞清楚Ubuntu安装过程中的分区，导致操作失误，导致把整个SSD给格了，Windows 10也因此没了。思来想去，考虑到Y450的硬件老化问题，以及无意再去搞破解、授权之类的乱七八糟的东西，所以下定决心使用Ubuntu。

<!-- more -->

从Windows迁移到Ubuntu并不是简单的事情，虽然之前也有Ubuntu使用经验，但是都是短期的，好多问题在解决的时候都是从网上的博客复制粘贴的命令行，有太多不稳定、不规范的因素。这次决定从Windows迁移到Ubuntu，一条很重要的原则是一切以官方文档为准。所以那些乱七八糟的博客的命令行尽量避开。除此之外，Ubuntu必定有很多跟Windows不一样的体验，所以另一条原则是不装模拟Windows运行环境的部件和软件。
在这两条原则下的迁移，主要是用户习惯和思维的转变。用户习惯主要是说从鼠标-图形界面操作过渡到命令行为主的操作；思维转变，主要是指摒弃在Windows系统下养成的各种陋习，搞破解、玩游戏、上qq等，由此转变为项目、任务为主、目标明确的高效学习和开发习惯。
下面以装完Ubuntu之后应该做的几件事为例，详细介绍这些原则和转变的体现。

# 更新系统
一般系统都会在发布之后，不断的打补丁，Ubuntu也不例外，即使是最新的系统。所以装完系统之后，第一件事是更新系统。更新的好处是去掉无用的库，添加新的库，为后面的软件安装排除故障。有几次在没更新系统的情况下装软件，出现了很多奇怪的Bug，更新系统后则没出现任何问题。
执行更新的命令很简单，分两步，首先是更新源（开源的系统的一大弊病就是源太多，而且各不相干，各自维护，不像Windows的更新源全由微软管理维护）：
```
sudo apt-get update
```
这句话必须在add-apt-repository之后执行，以保证后续行为的正常进行。也就是说在Ubuntu下装软件的完整步骤为：添加源->更新源->下载并安装软件。有些时候你不用执行添加源和更新源，那是因为你在很早之前就已经执行了这些操作。
第二步，更新系统：
```
sudo apt-get upgrade
```

# 装Chrome浏览器
Ubuntu下装Chrome比在Windows下装更容易，两步即可：下载软件包，双击安装。
下载x86版本：
```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_i386.deb
```
下载x64版本：
```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```
这里不用管具体的Chrome版本，因为stable_current就是“当前稳定版”的意思。
之所以装Chrome，是因为Chrome的书签管理、插件、审查元素等相关的用户习惯已经基本养成，不想再花时间在这些上面，Chrome的账户同步（包括插件和书签，以及一些基本设置）也是特别贴心。

# OpenVPN
之前写过一片博文讲[如何使用VPS和OpenVPN进行翻墙](http://kunsland.github.io/blogs/2015/03/22/vps-openvpn/)。这里就只讲客户端的安装，Ubuntu下安装OpenVPN不用翻墙，只需一个命令就行：
```
sudo apt-get install openvpn
```
然后将客户端配置文件xxx.conf，根证书，账户证书和账户私钥这四个文件拷贝到`/etc/openvpn/`目录下。
启动openvpn服务：
```
sudo service openvpn start
```
启动成功会在末尾行显示`Autostarting VPN 'xxxx'`。
设置开机自启动：
```
sudo vi /etc/default/openvpn
```
根据该文件注释的前几行的介绍，自启动有两种方式设置:一是去掉`AUTOSTART="all"`前的注释符号`#`；二是添加一行`AUTOSTART="xxx"`，这里xxx需要替换为你的.conf文件名，不加文件后缀。

# 远程软件Remmina
Ubuntu自带Remmina，但是在远程时不能远程声音，所以要对Remmina进行更新，可以参见[官方Wiki](https://github.com/FreeRDP/Remmina/wiki)。
具体操作如下：
```
sudo apt-add-repository ppa:remmina-ppa-team/remmina-next
sudo apt-get update
sudo apt-get install remmina remmina-plugin-rdp libfreerdp-plugins-standard
```
# Ubuntu下的中文输入法
我使用Ubuntu的还有一条原则是英文优先，所以系统的所有提示都是英文显示。但工作生活中免不了要使用中文，所以需要装中文输入法。选来选取最后还是用了搜狗输入法。比较好的是搜狗输入法有专门的[Linux版本](http://pinyin.sogou.com/linux/)，并且还在更新当中。不过安装过程稍微有些折腾。首先是从官网下载deb安装包，双击即可安装。但是安装完之后还需要一定配置才能使用。
* 在将键盘输入法的系统从iBus改为fcitx（Language Support->Language->Keyboard input method system）
* 安装中文语言（Install/Remove Languages，勾选Chinese(Simplified)）
* 将Language for menus and windows中的`汉语(中国)`拖到第一的位置（覆盖掉第一个），并应用到整个系统（Apply System-Wide）
* 重启系统，配置文本输入（Text Entry）添加搜狗拼音输入法

如果你不喜欢英文界面的系统，就到此为止了；如果你喜欢英文界面的系统，就将Language for menus and windows中的`汉语(中国)`拖到末尾并应用到整个系统即可。

搜狗输入法会在某些时候失效，不起作用，比如在Emacs中不仅不起作用，而且切换快捷键跟Emacs的快捷键起冲突。解决办法：
安装zh_CN.utf8字符集：
```
sudo dpkg-reconfigure locales
```
接着设置LC_CTYPE="zh_CN.UTF-8"
```
sudo vi /etc/default/locale
```
这样配置可以解决搜狗输入法在Emacs中失效和热键冲突两个问题。

# Node.js
推荐如下安装方法：
```
sudo apt-get install nodejs
sudo apt-get install npm
ln -s /usr/bin/nodejs /usr/bin/node
```

# git
```
sudo apt-get install git
git config --global user.name="xxx"
git config --global user.email="xx@ss.yy.com"
ssh-keygen -t rsa -b 4096 -C "xx@ss.yy.com"
```
更多查看[git官方文档](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)以及[github官方文档](https://help.github.com/articles/generating-ssh-keys/)。

# Emacs
准备工作
```
sudo apt-get install build-essential
sudo apt-get build-dep emacs24
```
从[Emacs官网](http://gnu.mirror.iweb.com/emacs/)下载源码并解压，进入解压后的目录执行：
```
./configure
make
sudo make install
```
创建启动器：
```
sudo gedit /usr/share/applications/Emacs.desktop
```
文本中添加如下内容：
```
[Desktop Entry]
Version=1.0
Name=Emacs-24
Exec=env UBUNTU_MENUPROXY=0 /usr/local/bin/emacs
Terminal=false
Icon=emacs
Type=Application
Categories=IDE
X-Ayatana-Desktop-Shortcuts=NewWindow
[NewWindow Shortcut Group]
Name=New Window
TargetEnvironment=Unity
```
Emacs使用说明书，参见[Emacs官网](http://www.gnu.org/software/emacs/manual/emacs.html)。

# Latex
```
sudo apt-get install texlive-full
```
[LaTex官网](https://latex-project.org)，[入门指导](http://www.andy-roberts.net/misc/latex/)。