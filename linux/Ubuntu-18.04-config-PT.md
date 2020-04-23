# Ubuntu 18.04 小型主机挂PT配置
#### <p align="right"> 作者：BG2DGR</p>
[TOC]

## 软硬件情况

### 硬件

- 主机: Intel Up Square (x86平台, 嵌入式主机)

- 硬盘: 笔记本机械硬盘+硬盘盒(机械硬盘已经分区, 硬盘盒为usb3.0接口)

- 外部网络: 学校有线网络使用PPPoE拨号上网(有线拨号才支持IPv6)

- 内部网络: 设备通过千兆交换机连接到校园网

  ```
  校园网 <-> 千兆以太网交换机 <-> Up Square 主机 <-> 移动硬盘
  ```

### 软件

系统: Ubuntu 18.04

PT站: 北邮人PT(IPv6 only)

PT软件: deluge

局域网共享: Samba

***

***

## 配置指南

### 1. 禁止系统自动挂载移动设备

因为我的硬盘已经分区, 只有一个100G的分区用于PT站, 所以Ubuntu默认会在系统登录的时候, 挂载全部的移动设备, 这样导致不用的分区也被挂载.  

因此, 需要首先设置移动设备的自动挂载情况.

> 参考文献:  
>
> [How to disable GUI desktop USB automount on Linux System]: https://linuxconfig.org/how-to-disable-gui-desktop-usb-automount-on-linux-system



#### 1.1 安装` dconf-editor `

```shell
$ apt-get install dconf-editor
# 运行
$ dconf-editor
```

![安装dconf-editor](Ubuntu-18.04-config-PT\install-dconf-editor.png)

#### 1.4 关闭automount

在左侧导航栏定位到:

```
org->GNOME->desktop->media-handling
```

因为Ubuntu18.04默认使用GNOME桌面,(如果你使用其他桌面系统, 请自行更改)

![关闭automount](Ubuntu-18.04-config-PT\disable-automount.png)

如上图, 关闭automount和automount-open

### 2. 设置自动挂载移动硬盘的指定分区

上文已经介绍了取消自动挂载全部分区, 但是为了实现开机自动挂PT, 需要系统能自动挂载PT分区.

系统实现自动挂载有以下几种途径:

1. 编辑/etc/fstab
2. 编写脚本(mount命令), 并将脚本加入开机启动当中
3. 使用systemd的mount功能, 编写mount unit文件

由于新的系统已经减少了类似fstab、service命令、local.rc等系统管理文件的使用, 现在新的系统更加倾向于使用systemd来管理系统, 因此博主采用第三种方案进行显示自动挂载的功能, 而且**通过第三种统一使用systemd进行管理启动项和挂载项, 可以保证在磁盘没有正确挂载的情况下不启动deluge服务, 以免产生错误**

> 参考文章
>
> [Mounting Partitions Using Systemd]: https://oguya.ch/posts/2015-09-01-systemd-mount-partition/
> [在ubuntu 18.04下设置开机自动挂载移动硬盘]: https://ywnz.com/linuxjc/1963.html	"这篇文章讲述了第一种方式挂载"

#### 2.1 查询磁盘分区的UUID

```shell
$ sudo blkid
```

会得到当前所以的磁盘情况, 我只截取了一条

```
/dev/sda2: LABEL="PT" UUID="7AF29A26F299E6A3" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="fbb4a097-ea64-4a09-8330-c4a20740b8b3"
```

我的卷标为PT的盘的UUID为 `7AF29A26F299E6A3`

#### 2.2 创建挂载文件夹

Ubuntu 18.04 默认在图形界面挂载磁盘的时候, 会挂载到`/media/YOURUSERNAME/磁盘卷标`文件夹下, 因此我们也这样建立我们的挂载文件夹

```shell
$ sudo mkdir -p /media/YOURUSERNAME/PT
```

#### 2.2 编写mount模块

创建mount模块文件

```shell
$ sudo vim /etc/systemd/system/media-YOURUSERNAME-PT.mount
```

**注意!!! 这个mount模块的文件名十分讲究, 必须与你的挂载路径有关, 如我的挂载路径是/media/YOURUSERNAME/PT, 那面我的mount文件名应该为media-YOURUSERNAME-PT.mount**

如果你采用了自己的有`-`连字符的特殊路径, 请自行查阅上述参考文章

输入以下内容

```
[Unit]
Description=Mount PT disk

[Mount]
What=/dev/disk/by-uuid/7AF29A26F299E6A3
Where=/media/YOURUSERNAME/PT
Type=ntfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

**注意!!! What 应该是你的UUID, Where应该是你上一步创建的文件夹路径**



### 3. 设置拨号上网

校园网使用PPPoE拨号上网, Ubuntu18.04的桌面系统就可以发起PPPoE连接, 但是这里使用一个更加方便 的拨号办法(图形界面不好找到位置, 而且不容易自动连接或者断线重连)

> 参考文献: 
>
> [ADSL（PPPOE）接入指南]: https://wiki.ubuntu.org.cn/ADSL（PPPOE）接入指南

#### 3.1 安装pppoeconf拨号软件

建议先检查系统中是否有pppoeconf命令

```shell
$ sudo apt-get install pppoeconf
```

#### 3.2 进行pppoeconf上网参数设置

pppoeconf提供了一个简单的基于命令行的"图形"配置接口.

```shell
$ sudo pppoeconf
```

运行之后

一个基于文本菜单的程序会指导你进行下面的步骤：

1. 确认以太网卡已被检测到。

2. 输入你的用户名（由ISP所提供 注意：输入时请先清除输入框中的“username“，否则可能造成验证错误）。

3. 输入你的密码（由ISP所提供）。

4. 如果你已经配置了一个PPPoE的连接，会通知你这个连接将会被修改。

5. 弹出一个选项:你被询问是否需要'noauth'和'defaultroute'选项和去掉'nodetach',这里选择"Yes"。

6. Use peer DNS - 选择 "Yes".

7. Limited MSS problem - 选择 "Yes".

8. 当你被询问是否在需要在进入系统的时候自动连接，你可以选择"Yes"。

9. 最后，你会被询问是否马上建立连接。

如果选择自动连接的话, 系统就会在登录的时候自动连接到PPPOE, 而且如果掉线也应该会重新连接(并没有测试).

### 4. 安装配置使用deluge

deluge是北邮人PT官方推荐的Linux下BT软件, 项目开源, 安装简单, 具有在命令行客户端基础上, 封装有守护进程(deluged) 图形界面前端(deluge) web前端(deluge-web), 不安装deluged和deluge-console的情况下, deluge会启动自身的内核模块, 当系统启动了deluged时, 图形前端会自动连接守护进程 

> 参考文献:
>
> [Ubuntu 安装 deluge 1.3.15 BT客户端]: https://www.24kplus.com/linux/338.html

#### 4.1 安装deluge

- 安装GTK程序

```shell
$ sudo apt-get install deluge
```

- 安装Web管理面板

```shell
$ sudo apt-get install deluged deluge-web deluge-console
```

deluge程序是基于Python的, 所以需要python支持(Ubuntu自带python)

#### 4.2 配置deluge

#### 4.3 设置开机自动启动

- 设置deluged后台程序的启动文件

```shell
$ sudo vim /etc/systemd/system/deluged.service
```

在文件中写入如下内容

```shell
[Unit]
Description=Deluge Bittorrent Client Daemon
Documentation=man:deluge
After=network-online.target media-YOURUSERNAME-PT.mount
[Service]
Type=simple
User=YOURUSERNAME
ExecStart=/usr/bin/deluged -d
ExecStop=/usr/bin/killall -w -s 2 /usr/bin/deluge
Restart=on-failure
TimeoutStopSec=300
[Install]
WantedBy=multi-user.target
```

**注意!!! 在After 里面有个 .mount模块, 是我们前面创建的开机挂载硬盘的模块, 请正确填写**

- 创建deluge-web 网页管理端的启动文件

```shell
$ sudo vim /etc/systemd/system/deluge-web.service
```

在文件中写入如下内容

```shell
[Unit]
Description=Deluge Bittorrent Client Web Interface
Documentation=man:deluge-web
After=network-online.target deluged.service
Wants=deluged.service
[Service]
Type=simple
User=YOURUSERNAME
ExecStart=/usr/bin/deluge-web
ExecStop=/usr/bin/kill /usr/bin/deluge-web
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

**注意!!!** 请将两个文件中User换为你自己的用户名, 否则将会出现权限问题, 导致deluge无法读取磁盘!!!



- 启动两个服务

```shell
$ sudo systemctl start deluged
$ sudo systemctl start deluge-web
```

- 查看服务状态

```shell
$ sudo systemctl status deluged
$ sudo systemctl status deluge-web
```

如果你看到类似的返回信息

```shell
● deluged.service - Deluge Bittorrent Client Daemon
   Loaded: loaded (/etc/systemd/system/deluged.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-10-31 16:24:38 CST; 6h ago
     Docs: man:deluge
 Main PID: 1037 (deluged)
    Tasks: 7 (limit: 2150)
   CGroup: /system.slice/deluged.service
           └─1037 /usr/bin/python /usr/bin/deluged -d
```

则说明运行成功了

**注意:**如果系统没有启动成功, 请检查以上两个文件的ExecStart 和ExecStop 的路径!!

- 设置开机启动

````shell
$ sudo systemctl enable deluged
$ sudo systemctl enable deluge-web
````

### 5. 设置Samba服务

最简单的实现Linux\Windows\OSX等系统之间的互传文件的方案就是使用Samba服务器.下面我们简单介绍如何配置samba服务

>参考文献: 
>
>[Ubuntu 18.04安装Samba服务器及配置]: https://www.linuxidc.com/Linux/2018-11/155466.htm

#### 5.1 安装Samba服务

```shell
$ sudo apt-get install samba samba-common
```

#### 5.2 设置Samba登录用户

```shell
$ sudo smbpasswd -a USERNAME
```

#### 5.3 设置配置文件

```shell
$ sudo vim /etc/samba/smb.conf
```

在配置文件smb.conf的最后添加下面的内容：

```shell
[share]
comment = share folder
browseable = yes
path = /media/YOURUSERNAME/PT/
create mask = 0700
directory mask = 0700
valid users = YOURUSERNAME
force user = YOURUSERNAME
force group = YOURUSERGROUP
public = yes
available = yes
writable = yes
```

**注意更改用户名用户组和分享路径**

更改完配置之后别忘记重启服务以生效配置

```shell
$ sudo systemctl restart smbd
```

#### 5.4 在Windows上访问

使用`WinKey+R` 召唤出"运行"窗口, 在其中输入

```
\\YOURIP
```

回车之后, 输入之前设置好的用户名和密码进行访问

#### 5.5 设置静态IP(内网)

我是用的是交换机通过pppoe上网, 但是交换机没有DHCP服务, 如果不手动配置内网的IP地址, 就无法通过交换机进行访问Samba服务器.

> 参考文献:
>
> [Ubuntu 18.04设置静态IP]: https://my.oschina.net/u/2306127/blog/1923450

- 首先查询自己的网卡信息

```shell
$ ifconfig
```

确定自己的以太网卡名称, 我的网卡名词为 `enp2s0`

- 设置网卡为静态IP

**Ubuntu 18.04 采用了新的网络管理方式, 但是我还是采用了编辑interface文件的方式进行静态IP设置**

```shell
$ sudo vim /etc/network/interfaces
```

在文件末尾, 修改为如下信息

```shell
auto enp2s0
iface enp2s0 inet static
address 192.168.1.111
netmask 255.255.255.0
gateway 192.168.1.1

dns-nameserver 114.114.114.114
```

### 6. 设置SSH服务器

有了SSH服务器之后, 就可以远程登录了, 而且如果的命令比较熟悉的话, 也可以通过SSH进行配置系统, 也就不用再往小主机上插入鼠标键盘显示器了!

> 参考文献
>
> [How to Enable SSH Server on Ubuntu 18.04 LTS]: https://linuxhint.com/enable_ssh_server_ubuntu_1804/

#### 6.1 安装SSH server

```shell
$ sudo apt-get install openssh-server
```

#### 6.2 设置开机启动

````shell
$ sudo systemctl enable sshd
````

#### 6.3 设置防火墙权限

首先查看防火墙是否启动

```shell
$ sudo ufw status
```

如果没启动, 无需设置下一步, 如果启动了, 则需要添加SSH的允许

```shell
$ sudo ufw allow ssh
```