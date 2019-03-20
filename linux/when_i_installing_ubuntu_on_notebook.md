# 数数我在安装ubuntu的时候踩的坑
#### <p align="right"> 作者：HITYJ<p>

------

## 0 写在前面
从昨晚到今天下午，我在安装Ubuntu的过程中踩了很多很多坑。一怒之下成此文

## 1 invidal magic number
在下载镜像，ultraiso刻录，U盘启动，选择install ubuntu之后
报错！
> invidal magic number

google之，无果
先怀疑是不是包出了问题于是使出sha-256校验大法，一模一样  
更换关键词再找，一段话令我印象深刻
> My fault for buying cheap generics off amazon[1]

不要用山寨货！！！  
扔下我刚买的几块钱便宜U盘，找舍友借了个闪迪，制作启动盘之后果然不出现这个错误了  
**便宜没好货**，古人诚不欺我


## 2 一屏幕的报错
这个阶段持续最长，折磨了我至少四个小时，包含
>ACPI Error  
>ACPI BIOS Error(bug)  
>usb usb*-port*: Cannot enable  

等奇奇怪怪的东西。
通过对`ACPI Error`、`ubuntu`和`18.04.2`等关键词的查找，得到了一些解决方法[2]
>选择 install ubuntu 的地方 按e   
>在linux开头那行末尾处 加入 acpi=off  

在知乎也找到了一些说法，在相同的地方添加 `nomodeset` ，然而根据我的实测，这一条没用  
之后就是一次次漫长的尝试以及自我否定了，即使添加了`acpi=off`这一项，卡在刷屏的文字报错，紫底的加载界面也是很正常的

## 3 SQUASHFS error
在无数次手动9秒关机再开机，f9选择U盘启动以及键入`acpi=off`之后，我来到了历史性的阶段，图形界面出现了！  
然后，在我选择`简体中文`并按下下一步之后，指针变成了圈圈，状态栏变成黑色，**它假死了！**
>沉默是今晚的康桥  --徐志摩

**retry**，新的进展发生在`简体中文`的下一步--选择键盘，这次它不是假死，是图形界面消失了！熟悉的黑底白字命令行出现在面前，此时我无比怀念我的蓝色powershell(windows下的一款shell)。定睛一看，报错如下
> reboot gnome failed(gnome是ubuntu18.04默认的桌面环境)  
>SQUASHFS error  

直觉使得我在bing输入框输入了“SQUASHFS”
>Squashfs（.sfs）是一套供Linux核心使用的GPL开源只读压缩文件系统。[3]

文件系统？莫非我借过来的U盘也是个寨货？  
搜索“SQUASHFS error”，出现的结果五花八门  
>你内存有毛病   
>你光驱坏了  
>你数据线有猫腻  
>你光盘不对劲  
>你下的iso坑了[4]  

莫非我用了11个月的内存条其实有暗病？（内存：想不到吧？）  
于是又换了个u盘，按照zzn的说法开了“刻录校验”，嗯，问题依旧。  
于是本头铁战士祭出了终极大招<u><b><em>retry</u></b></em>  
...  
成了，装上了，虽然liveCD不能重启但是这都是小问题。  
9s关机，一键开机.输入密码，enter！  
嗯？
怎么卡屏了

## 4 输入密码即卡屏
9s大法，一键开机，在grub(ubuntu自带的引导程序)里面再次复习一下内核添加`acpi=off`  
输入密码，enter！  
我成功装上了虚拟机里面一路下去重启一次就能装上的Ubuntu。  

**Amazing！**

## 5 不能关机和重启
同liveCD一样，我的Ubuntu现在不能用9s大法之外的方法关机或重启。  
经过google，方法来了：  
>sudo vim /etc/default/grub  GRUB_CMDLINE_LINUX="noacpi acpi=off acpi=force apm power_off=1"  
>sudo vim /etc/modules 加上一行 apm power_off=1  
>sudo update-grub[5]  

vim是GNU/linux的一款非常反人类的**文字编辑器**  
我们可以使用更好操作的gedit在此处代替vim  
shift + alt + t 快捷键呼出终端，以下sudo开头的东东都是指在终端内输入
>sudo gedit /etc/default/grub  
>在GRUB_CMDLINE_LINUX="..."引号内后面加上   
>>acpi=off acpi=force    
>
>保存，退出
>
>sudo gedit /etc/modules
>文件末尾加上一行
>>apm power_off=1  
>
>保存，退出  
>
>sudo update-grub

再次9s大法之后，重启进入Ubuntu。理论上以后就不需要9s大法来重启了  

## 6 双系统的时间差别

>先说下两个概念：  
>UTC即Universal Time Coordinated，协调世界时（世界统一时间）  
>GMT 即Greenwich Mean Time，格林尼治平时  
>Windows 与 Mac/Linux 看待系统硬件时间的方式是不一样的：Windows把计算机硬件时间当作本地时间(local time)，所以在Windows系统中显示的时间跟BIOS中显示的时间是一样的。  
>Linux/Unix/Mac把计算机硬件时间当作 UTC， 所以在Linux/Unix/Mac系统启动后在该时间的基础上，加上电脑设置的时区数（ 比如我们在中国，它就加上“8” ），因此，Linux/Unix/Mac系统中显示的时间总是比Windows系统中显示的时间快8个小时。  
>所以，当你在Linux/Unix/Mac系统中，把系统现实的时间设置正确后，其实计算机硬件时间是在这个时间上减去8小时，所以当你切换成Windows系统后，会发现时间慢了8小时。就是这样个原因。[6]

答主给出了两个解决方法，我使用了其中一种[6]  
>在Ubuntu中把计算机硬件时间改成系统显示的时间，即禁用Ubuntu的UTC。  
>这又有另一个需要注意的地方：在 Ubuntu 16.04 版本以前，关闭UTC的方法是编辑/etc/default/rcS，将UTC=yes改成UTC=no， 但在Ubuntu 16.04使用systemd启动之后，时间改成了由timedatectl来管理，所以更改方法是  
>> timedatectl set-local-rtc 1 --adjust-system-clock  
>
>执行后重启Ubuntu，应该就没有问题了。

另一种解决方法可自行查阅原文

在使用本方法后可能出现Windows时间仍然不对的情况，进入bios将时间往后调8个小时即可

## 7 后记

愿看到这篇文章的你并不需要用到这些东西  
good lock and have fun

## 参考
1:https://ubuntuforums.org/showthread.php?t=2410931  
2:http://www.cnblogs.com/handsomer/p/9310559.html  
3:https://zh.wikipedia.org/wiki/SquashFS  
4:https://help.ubuntu.com/community/SquashfsErrors  
5:https://bbs.csdn.net/topics/392066577  
6:https://www.zhihu.com/question/46525639  

[1]:https://ubuntuforums.org/showthread.php?t=2410931
[2]:http://www.cnblogs.com/handsomer/p/9310559.html
[3]:https://zh.wikipedia.org/wiki/SquashFS
[4]:https://help.ubuntu.com/community/SquashfsErrors
[5]:https://bbs.csdn.net/topics/392066577
[6]:https://www.zhihu.com/question/46525639