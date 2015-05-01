title: VPS+OpenVPN搭建VPN服务器
date: 2015-03-22 11:38:19
tags: [vps, openvpn, ipv6]
---

#安装OpenVPN最新版
```sh
yum install rpm-build openssl-devel lzo-devel pam-devel gcc
wget https://swupdate.openvpn.org/community/releases/openvpn-2.3.6.tar.gz
rpmbuild -tb openvpn-xxx.xxx.tar.gz
```

在rpmbuild/RPMS目录下（或者子目录下）找到openvpn-x.x.x.-xxx.rpm
```sh
rpm -ivh rpmbuild/RPMS/XXX/openvpn-x.x.x-xxx.rpm
```

<!-- more -->

#给vpn连接添加授权认证
```sh
wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
unzip master.zip
cd easy-rsa-master/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass #生成根证书，可忽略需要输入的信息，直接一路回车
./easyrsa build-server-full server nopass #服务器（server）密钥
./easyrsa build-client-full client-1 nopass #用户1（client-1）密钥
./easyrsa build-client-full client-2 nopass #用户2（client-2）密钥
./easyrsa build-client-full client-3 nopass #用户3（client-3）密钥
./easyrsa gen-dh #生成Diffie Hellman参数（DH文件）
```
这样就生成了一个根证书（其他证书的祖宗），一个DH文件，server端的密钥对，3个client的密钥对。根证书和DH参数放在`pki`目录下，公钥放在
`pki/issued/`目录下，私钥放在`pki/private/`目录下。
复制server密钥到/etc/openvpn目录下：
```sh
cp /root/easy-rsa-master/easyrsa3/pki/ca.crt /etc/openvpn
cp /root/easy-rsa-master/easyrsa3/pki/dh.pem /etc/openvpn
cp /root/easy-rsa-master/easyrsa3/pki/issued/server.crt /etc/openvpn
cp /root/easy-rsa-master/easyrsa3/pki/private/server.key /etc/openvpn
```

#从VPS上下载文件

使用pscp从VPS上下载用户证书(client-x.crt)、用户私钥(client-x.key)、VPS根证书(ca.crt)
在你的电脑上下载http://the.earth.li/~sgtatham/putty/latest/x86/pscp.exe
用命令行执行`pscp.exe root@<YOUR VPS IPV4 ADDR>:<SOURCE PATH> <TARGET PATH>`
例如：
```cmd
pscp.exe root@192.168.1.123:/root/test.txt ﻿D:/test.txt
```
最好把根证书和所有client的密钥对都下载下来，服务器的密钥对不用下下来。

#服务器端配置文件server.conf

在`/etc/openvpn/`目录下创建该文件。私网有三类：10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16。由于我所在的地方使用第一类局域网，所以我就将我的VPN私网设置为第三类（具体为192.168.234.0），以避免路由冲突。IPV6也是一样的道理，需要给链接VPN的电脑分配一个私网地址，这里用的IPV6私网网段为`2001:db8:0:123::`，如果跟你现有的IP有冲突就得换一个。
```sh
port 1194 #可自行更改，客户端配置文件的端口号要与此一致
proto udp #可改成tcp，但要与客户端配置文件设置一致
proto udp6 #如果vps支持IPV6，则添加此项。IPV6下的tcp为tcp6
dev tun
server 192.168.234.0 255.255.255.0 #VPN下的IPV4子网设置，即DHCP设置
server-ipv6 2001:db8:0:123::/64 #如果VPS支持IPV6，则vpn下IPV6子网设置
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1" #设置客户端全部流量走vpn
push "dhcp-option DNS 8.8.8.8" #可根据需要考虑是否保留
push "dhcp-option DNS 8.8.4.4"
ca ca.crt #根证书，务必保证所有配置文件的ca.crt都是同一个根证书，这是其它根证书的祖宗。
cert server.crt #服务器的密钥对
key server.key
dh dh.pem #服务器配置文件务必要设置此项，客户端则不用。
keepalive 10 120
cipher AES-256-CBC #传输加密方式，这是目前最流行、最安全的加密方式；务必与客户端一致；
comp-lzo
persist-key
persist-tun
status openvpn-status.log #日志文件
verb 3
```
#客户端配置文件client-1.conf
Windows下为client-1.ovpn，实际上都可以用，只不过ovpn后缀可以双击启动OpenVPN。
```sh
client #注意这里与服务器端配置文件不一样，服务器端配置文件不用设置为server
dev tun
proto udp
proto udp6 #如果你的网络支持IPV6
remote <YOUR VPS IPV4 ADDR> 1194
remote <YOUR VPS IPV6 ADDR> 1194
route 10.0.0.0 255.0.0.0 net_gateway #局域网不走vpn，以便局域网内远程、联机等。
resolv-retry infinite
nobind
ca ca.crt #根证书
cert client-1.crt #客户1（client-1）的密钥对
key client-1.key
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
comp-lzo
verb 3
```
#sysctl.conf和iptables设置
```sh
vi /etc/sysctl.conf
```
将`net.ipv4.ip_forward = 0`改为`net.ipv4.ip_forward = 1`；
让更改立即生效：
```sh
sysctl -p
```
执行完后可能会有error警告，可以忽略与`net.ipv4.ip_forward`无关的error警告。

非OpenVZ架构的VPS：
```sh
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 192.168.234.0/24 -j ACCEPT
iptables -A FORWARD -j REJECT
iptables -t nat -A POSTROUTING -s 192.168.234.0/24 -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -j SNAT --to-source <YOUR VPS IPV4 ADDR>
```

OpenVZ架构的VPS（包括bandwagon，berry.pw等）：
```sh
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -s 192.168.234.0/24 -j ACCEPT
iptables -A FORWARD -j REJECT
iptables -t nat -A POSTROUTING -o venet0 -j SNAT --to-source <YOUR VPS IPV4 ADDR>
iptables -t nat -A POSTROUTING -s 192.168.234.0/24 -j SNAT --to-source <YOUR VPS IPV4 ADDR>
iptables -t nat -A POSTROUTING -j SNAT --to-source <YOUR VPS IPV4 ADDR>
```

之后都要执行：
```sh
service iptables save
```
#VPN连接
服务器端执行：
```sh
service openvpn start
```
可将此命令添加到`/etc/rc.local`使openvpn开机自启动

__客户端__：去下载客户端[OpenVPN](https://openvpn.net/index.php/download/community-downloads.html)，并安装。
__Mac OS客户端__: 点我下载[Tunnelblick](http://sourceforge.net/projects/tunnelblick/files/latest/download?source=files)。
__Android客户端__：去应用市场搜OpenVPN，开发者为openvpn。
__IOS客户端__：也是去应用市场搜。

__PC端__：将ca.crt、 client-1.conf、 client-1.crt、 client-1.key 这4个文件拷贝到OpenVPN安装目录下的config文件夹下。 启动OpenVPN后，程序会自动扫描该目录，右键即可弹出连接选项，连接到VPN。__Windows客户端注意使用管理员权限__。

__移动端__：首先将上述4个文件放到你想放的任何位置，然后通过OpenVPN的Import选项指定你的.conf或者.ovpn文件，然后连接即可。


#如何让客户端的局域网请求不走vpn？
私网有三类：10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16。
第一类在client.config中添加:
```sh
route 10.0.0.0 255.0.0.0 net_gateway
```
第二类在client.config中添加：
```sh
route 172.16.0.0 255.127.0.0 net_gateway
```
第三类在client.config中添加：
```sh
route 192.168.0.0 255.255.0.0 net_gateway
```
懂掩码原理的可自行研究如何缩小范围。

#问题：Incoming packet rejected
* 解决方法一：
在server.conf中加上：`local <YOUR VPS IPV4/IPV6 ADDR>`
* 解决方法二：
将IPV6和IPV4的服务器配置拆开成两个文件，可能需要指定不同的端口。

#OpenVPN官方文档
[如何配置OpenVPN](https://openvpn.net/howto.html)
[如何让OpenVPN支持IPv6](https://community.openvpn.net/openvpn/wiki/IPv6)
