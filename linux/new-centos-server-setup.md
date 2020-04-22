
# 新建服务器之后要做的一系列安全工作
#### <p align="right"> 作者：BG2DGR</p>

[TOC]

很多同学购买了VPS服务器之后就随意使用了, 这之间往往存在很大的安全隐患, 本文通过对SSH, 账户, Firewall 以及 SELinux的设置, 以达到较高的安全性.
<!-- More -->
## 0x01 用户设置
系统: CentOS 7.5
### 1. 新建用户, 并授权
系统安装之后, 默认只有root用户, 首先使用root 登录, 并新建用户  
- 新建用户
    ```shell
    # adduser username
    ```
    PS ***"#"代表是root用户命令***

- 设置密码
    ```shell
    # passwd username
    ```

- 加入sudoers  
    加入可以使用sudo命令的用户组  
    a. 更改sudoers文件权限(由只读到可写)
    ```
    # chmod -v u+w /etc/sudoers
    ```
    b. 加入username用户到sudoers文件
    ```
    # vim /etc/sudoers
    ```
    在打开的文件中找到 "## Allow root to run any commands anywher", 并在root下面一行加入(vim 按下i键添加内容)
    ```
    ## Allow root to run any commands anywher  
    root    ALL=(ALL)       ALL
    username  ALL=(ALL)       ALL  #这个是新增的用户
    ```
    (vim 按esc 再按":wq"保存)
    c. 收回sudoers文件权限
    ```
    # chmod -v u-w /etc/sudoers
    ```

- 使用新用户登录, 并验证sudo权限
    ```
    $ sudo whoami
    root
    ```
    返回root, 则成功

## 0x02 SSH安全设置
### 1. 禁止root用户登录ssh
这是一件很危险的事情, 必须阻止!
```
$ sudo vim /etc/ssh/sshd_config
```
找到 并解除注释
```
PermitRootLogin no
```
重启服务
```
$ sudo systemctl restart sshd
```

### 2. 更改SSH默认端口
```
$ sudo vim /etc/ssh/sshd_config
```
找到 ```#port 22```
并改为:
```
port 22     # 要保留22端口(默认端口), 大概率新端口不生效
port 12345  # 你想要的随机端口号
```
重启服务
```
$ sudo systemctl restart sshd
```

尝试链接你新更改的端口 12345, 如果发现无法连接, 从以下几个方面考虑:  
- 防火墙是否开启  
    查看firewalld  
    ```$ firewall-cmd --state```  
    查看返回值, 多半没开  
    查看iptables  
    ```$ service iptables status```  
    查看返回值, 多半没开(iptables用于centos6之前, 而centos7之后采用firewall, firewall其实是iptables的一个封装,更方便管理, 之后我们会在服务器上启用它!)
- SELinux是否启动  
    我们ssh的新端口无法连接多半是这个问题:  
    首先查看SELinux是否开启  
    ```$ /usr/sbin/sestatus -v```  
    如果SELinux status参数为enabled即为开启状态,   
    使用以下命令查看当前SElinux 允许的ssh端口：
    ```
    $ sudo semanage port -l | grep ssh
    ```
    返回"ssh_port_t                     tcp      22"
    添加新端口到SELinux
    ```
    $ sudo semanage port -a -t ssh_port_t -p tcp 12345
    ```
    再次查看当前SElinux 允许的ssh端口：
    ```
    $ sudo semanage port -l | grep ssh
    ```
    如果成功会输出
    ```
    ssh_port_t                    tcp    12345, 22
    ```

再次尝试连接新的SSH端口, 成功则注释掉 ```port 22```, 重启SSH服务器重新登录22端口, 发现已经无法登录

### 3. 使用RSA密钥对登录
RSA是利用非对称加密的方式, 产生了一对公钥和私钥, 公钥放到服务器当中, 私钥放在自己的PC中, 每次登录的时候, 系统发送私钥到服务器, 通过公钥验证身份后, 建立安全链接. 通过这种方式登录服务器更安全, 更方便(密码登录可能你的密码不够复杂, 而且每次不需要你输入密码即可登录)  
- 生成密钥对  
    有很多中方法, 在Windows上, 可用git-bash内置的一些小工具生成, 或者用XShell软件自带的小工具生成, 本文使用git-bash内置小工具. 其他方式原理一致, 操作类似, 不再赘述  
    打开git-bash命令行工具, 并输入
    ```
    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key(~/.ssh/id_rsa) 
    ```
    这里你可以更改密钥产生的位置, 回车使用默认路径("~"代表windows下home环境变量对应值)
    ```
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    ```
    输入密码, 最好不为空

    之后会提示你生成成功, 并提示你保存的路径. 到达路径后发现生成的 'id_rsa' 和 'id_rsa.pub' 这个id_rsa.pub对应的是公钥,  id_rsa对应私钥.
- 服务端设置  
     把id_rsa.pub 上传到服务器, 并更改.ssh 路径权限
     公钥文件应该保存在你远程主机的用户主目录下的 `.ssh/authorized_keys `文件里
     ```
     $ chmod 700 .ssh
     $ cd .ssh
     $ ls
     ```
     进入之后查看是否存在 "authorized_keys"文件
     如果不存在, 则生成, 
     ```
     $ touch authorized_keys  
     ```
    如果存在则写入, (刚刚通过winscp将公钥文件上传到用户的home目录下)
    ```
    $ cat ~/id_rsa.pub >> ~/.ssh/authori_keyzeds
    ```
    更改authorized_keys文件访问权限
    ```
    $ chmod 400 ~/.ssh/
    ```

    设置ssh为RSA认证模式
    ```
    $ sudo vim /etc/ssh/sshd_config
    ```
    找到如下设置
    ```
    RSAAuthentication yes #RSA认证(我的系统配置里面没有这句, 可能需要自己加上)
    PubkeyAuthentication yes #开启公钥验证
    AuthorizedKeysFile .ssh/authorized_keys #验证文件路径
    PasswordAuthentication no #禁止密码认证 (这里建议确定可以RSA认证之后再更改为禁止密码登录)
    PermitEmptyPasswords no #禁止空密码
    ```
    重启ssh服务
    ```
    $ sudo systemctl restart sshd
    ```

## 0x03 防火前的配置
目前在CentOS 6以上的系统启用了新的firewall防火墙, 利用firewall防火墙可以更加严密的守护自己的系统
### 1. 安装防火墙
上文中已经提到了如何查看防火墙的状态, 多数情况下系统默认没有安装防火墙, 这时需要安装
```
$ sudo yum install firewalld
```
### 2. 启动防火墙
目前我还没有发现如果在防火墙不启动的时候添加防火墙规则, 所以前文我们将ssh端口更改, 并禁用了22端口, 这条我们先要撤销, (防火墙启动的时候默认只有22号端口启动). 并启动
```
$ sudo systemctl start firewalld
```
### 3. 设置防火墙规则
firewall是一个强大的防火墙, 有很多功能请各位自行探索, 这里只列出最基本的配置, 下面的命令能够让你"永久的"(permanent),添加一条规则, 这条规则开放了12345端口,  永久的意思就是当你重启firewall服务之后不会被撤销, (这其实是一个非常有用的设定, 当你错误的配置防火墙的时候,特别是禁止某些端口的时候, 不能永久生效让你可以靠重启服务器来纠正错误的配置)
```
$ sudo firewall-cmd --permanent --add-port=12345/tcp
```
### 4. 查看当前规则
```
$ sudo firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: ssh dhcpv6-client  # 这里ssh服务器对应的是22号端口,我们可以移除她
  ports: 12345/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
### 5. 移除规则
这里我们移除多余的ssh服务(22端口)
```
sudo firewall-cmd --permanent --remove-service=ssh
```
### 6. 生效规则
最紧张的时刻了, 如果配置错误的话, 可能你生效之后ssh就掉线了, 如果出错了可以跑到你服务器的控制台页面重启你的服务器
```
sudo firewall-cmd --reload
```
### 7. 开机启动firewall
如果你确定没有问题了, 可设置firewall为开机启动
```
sudo systemctl enable firewalld
```
至此你的服务器就比较安全了, 更多的有关firewall, systemd, ssh, rsa等相关内容请自行Google!!!

## 0x04 参考文献
>[CentOS 7中添加一个新用户并授权](https://www.linuxidc.com/Linux/2016-11/137549.htm)  
>[Centos7 修改SSH 端口](https://www.jianshu.com/p/c18d5347c9b6)  
>[CentOS 7配置SSH基于密钥对验证登录](http://pcvc.net/blog/2015/08/10/centos-7-configuring-ssh-key-based-authentication-login/)  
>[服务器VPS安全防护](https://zhuanlan.zhihu.com/p/26282070)
>[做好这两点，避免服务器成为肉鸡（傀儡）](https://zhuanlan.zhihu.com/p/56864040)  

