title: 关于搬瓦工(BandwagonHost)VPS
date: 2015-03-02 15:18:56
tags: [bandwagon, vps, ipv6, gfw]
---
之前一直使用fqrouter2翻墙与Google Play结合使用更新Nexus 5上的应用，直到新年之处（2015年1月中旬）发现fqrouter2不那么好使了，每次打开Google Play，能看到一大堆应用升级提示，但就是下载不了。偶然点进fqrouter2“帮助”（或是“关于”，具体记不清了）里的测试组链接，这个测试组是建在Google Plus上的，里面有大量关于fqrouter2的测试讨论，以及大量的关于翻墙手段的讨论。于是发现了一个名叫Anonymous V的牛人写的[使用Shadowsocks翻墙的教程](https://plus.google.com/103234343779069345365/posts/MLy82ud6wjY)。作者对VPS提供商的比较与选择、Shadowsocks的安装和配置这两大方面进行了详细的说明和指导。最终我结合自己的需求和财力，购买了搬瓦工VPS，搭建了ShadowSocks服务器（简称ss）。今天又花了一点时间弄PC端和手机端的配置，让SS同时支持IPV4和IPV6，速度一般，但已经满足我的需求了。


<!-- more -->


### 为什么选择搬瓦工？

__便宜__。我选的是Anonymous V提供的4.99美元一年的[套餐](https://bandwagonhost.com/aff.php?aff=1285&pid=20)，HDD: 2.0 GB, RAM: 96 MB, CPU: 1x Intel Xeon, 流量: 200 GB/月，物理地址在西海岸洛杉矶（LA）。

### 有了VPS能做些什么？
* 手机、电脑翻墙，同时支持IPV4/IPV6
* 使用Google Play给手机更新APP
* 通过建立IPV6代理，减少校园网收费所带来的费用

### 选择何种方式支付？
选择Paypal支付是最通用的方式，因此最好注册一个[Paypal账号](https://www.paypal.com/c2/webapps/mpp/home)，往里面充值或绑定银行卡/借记卡。
方式一：注册一个Paypal账号，绑定银行卡（我的Paypal->用户信息->财务信息->银行账户->添加我的银行），购买VPS时选择Paypal支付即可。
方式二：对于那些不放心直接将银行卡绑定到Paypal的，可以到[全球付](http://www.globalcash.hk/)注册一个帐号，可以用国内通用的支付方式（支付宝、网银等）买一张虚拟借记卡（MasterCard），这张虚拟卡同样也可绑定到Paypal（我的Paypal->用户信息->财务信息->借记卡和信用卡），再用Paypal支付即可。

### 配置ShadowSocks
参见Anonymous V写的[教程](https://plus.google.com/103234343779069345365/posts/Xce4EJpLGhX)。翻不了墙看我的简短版：
* 下载[putty](http://www.putty.org)，远程登录到VPS，搬瓦工的密码：My Services->KiviVM Control Panel->Root password modification->Generate xxxx。
* 安装必备部件，需要选择的时候，输入y回车：
```
yum install epel-release
yum update
yum install python-setuptools m2crypto supervisor
easy_install pip
pip install shadowsocks
```
* 在VPS上配置ShadowSocks服务器端，输入`vi /etc/shadowsocks.json`命令，按i进入编辑模式，输入或粘贴（Putty里单击右键为粘贴）如下内容（其中password是由你指定的任意密码，用来连接ShadowSocks）：
```
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"xxxxx",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
* 编辑好后，按ESC退出编辑模式，再按`SHIFT zz`（一个Shift键加两次z键）保存并退出文件编辑。
* 执行命令`vi /etc/supervisord.conf`，按i进入编辑模式，输入或粘贴如下内容：
```
[program:shadowsocks]
command=ssserver -c /etc/shadowsocks.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/shadowsocks.log
```
* 编辑好后，按ESC退出编辑模式，再按`SHIFT zz`（一个Shift键加两次z键）保存并退出文件编辑。
* 执行命令`vi /etc/rc.local`，按i进入编辑模式，文件末尾输入或粘贴如下内容：
```
service supervisord start
```
* 编辑好后，按ESC退出编辑模式，再按`SHIFT zz`（一个Shift键加两次z键）保存并退出文件编辑。
* 执行`reboot`命令，Putty会提示连接断开，隔几秒后可根据需要重连。
至此，ShadowSocks服务器端配置完毕，PC客户端可以下载[shadowsocks-gui](http://sourceforge.net/projects/shadowsocksgui/files/dist/)，智能手机客户端各大应用商城搜索ShadowSocks（又叫“影梭”），输入你的VPS IP地址和ShadowSocks密码即可，加密方式选aes-256-cfb。

### 给搬瓦工添加IPV6支持
搬瓦工不直接支持IPV6访问，参考网上搜到__CHON__的一篇[博客](http://ichon.me/post/659.html)，我给自己的搬瓦工VPS（刚好就是OpenVZ架构）添加了IPV6支持。

一般情况下，只需要在[HE Free IPv6 Tunnel Broker](https://tunnelbroker.net/)（简称__HETB__）注册一个帐号，登录之后用VPS的IPV4地址Create Regular Tunnel（地点选VPS的物理地址，我的是LA），然后根据Example Configurations的指导配置VPS即可。

但是搬瓦工的默认操作系统为centos 6，Example Configurations里面没有centos的配置指导，所以得进行如下操作：
* 检测tun/tap设备，运行如下命令，看是否返回`File Descriptor in bad state`，如果是则继续，不是则我也不知道该怎么弄（应该不会发生后面这种情况）。这里跟__CHON__博客中的一样。
```
cat /dev/net/tun
```
* 如果系统没有gcc（一般都没有gcc），需要安装gcc，这是__CHON__没提到的。
```
yum groupinstall "Development Tools"
```
* 下载并编译tb-tun，跟__CHON__博客中的一样
```
wget http://tb-tun.googlecode.com/files/tb-tun_r18.tar.gz
tar xvf tb-tun_r18.tar.gz
gcc tb_userspace.c -l pthread -o tb_userspace
```
* 给VPS添加IPV6隧道，跟__CHON__博客中的略有区别，最后一句是根据__CHON__的这篇博文下面的讨论提取出来的。如果卡在第一条命令，按回车可以继续进行后面操作。
```
setsid ./tb_userspace tb 5.6.7.8 1.2.3.4 sit
ifconfig tb up
ifconfig tb inet6 add 2001:a:b:c::2/64
ifconfig tb mtu 1480
route -A inet6 add ::/0 dev tb
ip -6 route del default dev venet0
```
* 运行`ping6 he.net`，看是否超时，如果超时则配置不成功；如果不断返回数据报响应延迟数据报告，则配置成功。按`Control + C`终止命令。
* __开机自动添加IPV6隧道脚本__，这里的脚本是根据__CHON__博文下面的讨论内容和自己的实践总结出来的。用`vi /etc/rc.local`命令在/etc/rc.local文件中添加如下内容。__注意第一行的区别__
```
/root/tb_userspace tb 5.6.7.8 1.2.3.4 sit &
ifconfig tb up
ifconfig tb inet6 add 2001:a:b:c::2/64
ifconfig tb mtu 1480
route -A inet6 add ::/0 dev tb
ip -6 route del default dev venet0
ifconfig tb up
```
* 配置VPS的ShadowSocks服务器端，将`/etc/shadowsocks.json`文件中的server对应的ip地址换成`::`，表示任意IP协议的任意地址。
* 配置ShadowSocks的客户端，将服务器 ip换成2001:a:b:c::2即可，其他不变。
* 注意以上IP地址要换成自己的IPV6 Tunnel Endpoints地址（[HETB](https://tunnelbroker.net/)->Main Page->[xxxx.ipv6.he.net]->），5.6.7.8为Server IPv4地址，1.2.3.4为Client IPv4地址（即VPS地址），2001:a:b:c::2/64为Client IPV6地址。

### 关于ShadowSocks全局代理
在勾选全局代理后，PC端的应用大都不会（IE浏览器除外）自动使用代理上网。以QQ为例，需要在登录时设置代理。也可以在网上搜索全局代理的解决方法，我试过Proxifier，感觉速度有影响。

### 让Windows系统更新也走代理
Windows系统下“全局代理”并不是真正的全局代理，系统更新(Windows Update)并没走SS代理。需要以__管理员身份__执行以下命令：
```
netsh winhttp set proxy localhost:1080 "<local>"
```
如果不想用代理了，记得重置winhttp：
```
netsh winhttp reset proxy
```