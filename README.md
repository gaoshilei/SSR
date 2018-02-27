谷歌2016年出了一个基于Linux内核的 BBR 拥塞控制算法，虽然咱不懂咋回事，还是大概知道它也是通过优化 TCP 底层协议来实现网速加速，跟锐速的原理一样。我是通过搬瓦工的 KiwiVM 后台安装的，用了之后感觉不咋地，网上查了一下，大部分网友都说锐速的加速效果要好于BBR。锐速已经停止新用户注册和维护了，之前虽然收费，但是现在已经有破解版。BBR不好用，有大神就开始魔改了，出了一个魔改版的BBR，据说效果比锐速还好！  
下文对于锐速和BBR魔改版安装都有介绍，你可以尝试一下，看哪个速度更快。  
下文的环境基于：CentOS6 X6

# 1、准备 VPS

[搬瓦工VPS](https://bwh1.net/aff.php?aff=10505)是我目前使用过性价比较高的 VPS，速度也比较稳定。  
[Vultr](https://www.vultr.com/?ref=7260141) 也是一个比较不错的选择，不过没有搬瓦工稳定。当然其他海外的VPS也都是可以的，至于选哪个这个就根据个人喜好来选择吧。  
**如何购买比较稳定的 VPS 可以参考我之前写的[VPS搭建高速SS服务器](https://www.gaoshilei.com/2016/05/19/VPS/)**   

<!-- more -->

由于天朝的网络环境，之前的 SS 虽然营运商破解不了协议，但是通过流量监测和干扰的方式会严重影响我们的访问网络，最明显的特征就是响应慢，网络卡。所以 SS 现在已经有很大的局限性了，SSR + 加速 应该是我们最好的选择。下文会介绍如何加速，如何安装 SSR 服务。   

或许有的同学会问 SS 和 SSR 有什么区别？  
答：SSR 只是在 SS 的基础上增加了一些新的协议和混淆功能，抗检测和干扰能力更强，简单点说如果你所在的地区没有运营商干扰你访问墙外网站，那么是没有区别的，如果你所在地区的国际出口宽带流量被检测和干扰的很严重，SSR 这个时候优势就体现出来了。不过就目前的形势来看，SS 被干扰已经是很普遍的情况了，个人建议安装 SSR，然后设置好协议和混淆，干扰不严重可以不用混淆，后面会有介绍。  

你购买或者配置VPS过程中遇到什么问题，可以通过邮箱 goslei1315@gmail.com 联系我，工作日一般2个小时之内回复，询问之前请描述清楚你遇到的问题。

**注意：购买 VPS 要选择 KVM 虚拟的，不要选择 OpenVZ 虚拟，后面你就会知道原因。**


# 2、安装锐速破解版  
## 1、执行脚本安装
首先介绍一下锐速是干嘛的，下面的介绍来自Google
> 锐速serverspeeder是一款TCP网络加速软件，能在Linux系统和Windows系统的服务器中安装，安装后能启到提高网络连接稳定性、带宽利用率、低访问失败率等作用，从而提高服务器网络访问速度。锐速并非实际增大服务器带宽，只是提高网络的稳定性和利用率而已。一个明显变化就是在同一VPS安装科学上网工具观看YouTube，没安装锐速前观看YouTube 720P视频非常不流畅，经常会出现缓冲现象；而安装锐速后能流畅观看YouTube 720P视频。  

由于锐速只支持 KVM 虚拟化的主机，OpenVZ 主机无法使用，购买时请注意分辨。   

先把一键安装的脚本下载下来：  

```
[root@California_VPS ~]# wget -N --no-check-certificate https://raw.githubusercontent.com/wn789/serverspeeder/master/serverspeeder.sh
```

如果报错 wget 命令找不到，需要安装  

```
[root@California_VPS ~]# yum -y install wget
```

脚本下载完成之后赋权执行：  

```
[root@California_VPS ~]# chmod +x serverspeeder.sh
[root@California_VPS ~]# bash serverspeeder.sh
```

如果看到下面这个，证明当前 VPS 系统内核不支持  

> This kernel is not supported. Trying fuzzy matching...
Serverspeeder is not supported on this kernel! View all supported systems and kernels here: https://www.91yun.org/serverspeeder91yun  

需要手动修改内核，或者重新安装系统（注意！CentOS7无法切换至更低内核，请务必使用CentOS6级别内核，2.6.32-XXX.el6.x86_64测试通过。[错误详情](https://www.centos.org/forums/viewtopic.php?t=57922)）， 由于我的系统内核不支持，所以要手动修改。 如果很不幸你看到了上面的报错，请直接跳到`2、手动修改内核`，如果你没有看到上面内容，而是看到了   

```
[Running Status]
ServerSpeeder is running!
version              3.10.61.0

[License Information]
License              B4C10AE5B485C0CE (valid on current device)
MaxSession           unlimited
MaxTcpAccSession     unlimited
MaxBandwidth(kbps)   unlimited
ExpireDate           2034-12-31

[Connection Information]
TotalFlow            1
NumOfTcpFlows        1
TotalAccTcpFlow      0
TotalActiveTcpFlow   0

[Running Configuration]
accif                eth0       
acc                  1
advacc               1
advinacc             1
wankbps              10000000
waninkbps            10000000
csvmode              0
subnetAcc            0
maxmode              1
pcapEnable           0
```

恭喜你，锐速已经安装完成。  

锐速常用的命令  

```
service serverSpeeder start #启动
service serverSpeeder stop #停止
service serverSpeeder reload #重新加载配置
service serverSpeeder restart #重启
service serverSpeeder status #状态
service serverSpeeder stats #统计
service serverSpeeder renewLic #更新许可文件
service serverSpeeder update #更新
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f #卸载
```


## 2、手动修改内核（非必须）  

注意在搬瓦工的后台面板（KiwiVM）查看你现在的系统类型，如后面带有 bbr 字样的，需要重新安装不带 bbr 的，否则你将无法成功修改内核   
![](http://oeat6c2zg.bkt.clouddn.com/newOSForSSR.png)

查看当前的系统内核版本：  

```
[root@California_VPS ~]# uname -r
2.6.32-642.el6.x86_64
```

锐速支持的 CentOS6 内核版本为 2.6.32-504.3.3.el6.x86_64，下面就要开始修改内核了，准备好内核文件执行安装：  

```
[root@California_VPS ~]# rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-firmware-2.6.32-504.3.3.el6.noarch.rpm
[root@California_VPS ~]# rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-2.6.32-504.3.3.el6.x86_64.rpm --force
```

可能要等个几分钟，全部完成之后查看是否安装成功：  

```
[root@California_VPS ~]# rpm -qa | grep kernel

kernel-firmware-2.6.32-642.el6.noarch
dracut-kernel-004-409.el6.noarch
kernel-2.6.32-504.3.3.el6.x86_64
kernel-2.6.32-642.el6.x86_64
```

执行 `pm -qa | grep kernel` 命令之后可以看到锐速支持的 kernel-2.6.32-504.3.3.el6.x86_64 内核已经安装完成。最后一步，确认内核已经被替换。  
重启 VPS，然后查看当前的系统内核  

```
[root@California_VPS ~]# uname -r
2.6.32-504.3.3.el6.x86_64
```

内核已经成功被替换成锐速支持的内核，可以继续第一步的安装工作了。  

# 3、安装 BBR 魔改版

**注意：因为内核问题 BBR 与速锐只能二选一。**  

安装魔改版BBR最好选择Debian/Ubuntu系统，因为这两个系统大佬制作了一键安装脚本，而且已经将库编译好了，如果用CentOS需要自己安装gcc4.9+，编译要很长时间，不建议在CentOS安装。下文的介绍主要还是在CentOS环境中操作，Debian/Ubuntu 有一键脚本非常傻瓜，下文也会有介绍。  

BBR 也只支持 KVM 虚拟化的主机 ，所以选主机一定要选 KVM 虚拟化的。  

## CentOS
###  1、升级内核  

安装魔改BBR的系统要求是4.10以上版本的kernel及对应的linux-header，gcc版本应在4.9以上，如果对应的内核版本链接失效了，可以去`http://elrepo.org`上找镜像网站，镜像网站里面有归档，可以找到对应4.12版本的内核，`http://www.jules.fm/elrepo/archive/`就是其中一个镜像网站。**BBR魔改版只能支持到4.12版本的内核，之前用4.13内核失败了**   

依次执行下面的命令  

```
[root@California_VPS ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@California_VPS ~]# yum install -y http://www.jules.fm/elrepo/archive/kernel/el6/x86_64/RPMS/kernel-ml-4.12.10-1.el6.elrepo.x86_64.rpm
[root@California_VPS ~]# yum remove -y kernel-headers
[root@California_VPS ~]# yum install -y http://www.jules.fm/elrepo/archive/kernel/el6/x86_64/RPMS/kernel-ml-headers-4.12.10-1.el6.elrepo.x86_64.rpm
[root@California_VPS ~]# yum install -y http://www.jules.fm/elrepo/archive/kernel/el6/x86_64/RPMS/kernel-ml-devel-4.12.10-1.el6.elrepo.x86_64.rpm
```


###  2、修改启动引导  

执行完成之后，修改启动引导，修改配置文件：  

```
[root@California_VPS ~]# vim /etc/grub.conf
```

可以看到启动项的配置文件，我们刚才添加的内核在第一个，所以要修改默认的启动内核  

```
 grub.conf generated by anaconda
#
# Note that you do not have to rerun grub after making changes to this file
# NOTICE:  You have a /boot partition.  This means that
#          all kernel and initrd paths are relative to /boot/, eg.
#          root (hd0,0)
#          kernel /vmlinuz-version ro root=/dev/sda2
#          initrd /initrd-[generic-]version.img
#boot=/dev/sda
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (4.12.10-1.el6.elrepo.x86_64)
        root (hd0,0)
        kernel /vmlinuz-4.12.10-1.el6.elrepo.x86_64 ro root=UUID=971ffe7e-0c71-40c1-97a9-bdb6b167d4b7 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM consoleblank=0 vga=0x305
        initrd /initramfs-4.12.10-1.el6.elrepo.x86_64.img
title CentOS (4.13.10-1.el6.elrepo.x86_64)
        root (hd0,0)
        kernel /vmlinuz-4.13.10-1.el6.elrepo.x86_64 ro root=UUID=971ffe7e-0c71-40c1-97a9-bdb6b167d4b7 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM consoleblank=0 vga=0x305
        initrd /initramfs-4.13.10-1.el6.elrepo.x86_64.img
title CentOS (2.6.32-642.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-642.el6.x86_64 ro root=UUID=971ffe7e-0c71-40c1-97a9-bdb6b167d4b7 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM consoleblank=0 vga=0x305
        initrd /initramfs-2.6.32-642.el6.x86_64.img
title CentOS 6 (2.6.32-642.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-642.el6.x86_64 ro root=UUID=971ffe7e-0c71-40c1-97a9-bdb6b167d4b7 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM consoleblank=0 vga=0x305
        initrd /initramfs-2.6.32-642.el6.x86_64.img

```

修改 default=0 保存，重启服务器，然后查看内核是否修改成功  

```
[root@California_VPS ~]# uname -r
4.12.10-1.el6.elrepo.x86_64
```

如果显示是刚才我们安装的内核证明已经修改成功，如果还是旧的内核，需要重新安装。

###  3、编译安装  

先安装基础版本的 gcc 和 gcc++

```
[root@California_VPS ~]# yum install gcc gcc++
```

Linux 默认安装的 gcc 版本是4.4.7，而编译魔改 BBR 的 gcc 版本至少要 4.9，所以要手动升级 gcc  

#### 1、升级 gcc 版本 

因为需要4.9以上的gcc版本，所以我们升级gcc版本到4.9.4，有人要问为什么不升级到最新版，我的原则是够用就行，你也可以到`http://ftp.gnu.org/gnu`查找最新版然后升级下载对应的包就可以了。  

##### 1、 下载安装包：  

```
[root@California_VPS ~]# wget http://ftp.gnu.org/gnu/gcc/gcc-4.9.4/gcc-4.9.4.tar.bz2
```

然后解压：

```
[root@California_VPS ~]# tar -jxvf gcc-4.9.4.tar.bz2
```

解压主要消耗CPU性能，搬瓦工的低配VPS都是单核的，这个包将近90MB，所以要比较久大概等个十分钟左右就好了。  

##### 2、下载供编译需求的依赖库

下载、配置安装依赖库，解压好的文件里面有安装脚本，依次执行下面的命令   

```
[root@California_VPS ~]# cd gcc-4.9.4
[root@California_VPS gcc-4.9.4]# ./contrib/download_prerequisites
```

##### 3、新建目录供编译存放文件

首先新建一个文件夹用来存放编译的文件  

```
[root@California_VPS gcc-4.9.4]# mkdir gcc-build-4.9.4  
[root@California_VPS gcc-4.9.4]# cd gcc-build-4.9.4/
```

#####  4、生成 Makefile 文件  

```
[root@California_VPS gcc-build-4.9.4]# ../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
```

#####  5、编译gcc  

```
[root@California_VPS gcc-build-4.9.4]# make
```

这一步时间比较长，大概需要一个多小时（取决于你的CPU性能）。不要重复编译，因为编译期间CPU的使用率基本上都是100%，因为之前装错了内核，后面又编译了一次，于是我在KiwiVM后台看到了这样的提示  
![](http://oeat6c2zg.bkt.clouddn.com/5272396EC1518BCC63AA0092F77C2D44.jpg)     
CPU长时间处于满负荷状态，把我的CPU时钟限制了。    
![](http://oeat6c2zg.bkt.clouddn.com/QQ20171109-110554@2x.png)   
所以 CentOS 安装 BBR魔改还是要谨慎啊，gcc4.9 编译太费CPU了。


#####  6、安装gcc

```
[root@California_VPS gcc-build-4.9.4]# make install
```

安装完成之后重启服务器，然后查看gcc版本号  

```
[root@California_VPS ~]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-unknown-linux-gnu/4.9.4/lto-wrapper
目标：x86_64-unknown-linux-gnu
配置为：../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
线程模型：posix
gcc 版本 4.9.4 (GCC)
```

升级成功，下面开始正式编译BBR。

#### 2、编译安装 BBR

```
[root@California_VPS ~]# wget -O ./tcp_tsunami.c https://gist.github.com/anonymous/ba338038e799eafbba173215153a7f3a/raw/55ff1e45c97b46f12261e07ca07633a9922ad55d/tcp_tsunami.c
[root@California_VPS ~]# echo "obj-m:=tcp_tsunami.o" > Makefile
[root@California_VPS ~]# make -C /lib/modules/$(uname -r)/build M=`pwd` modules CC=/usr/bin/gcc
[root@California_VPS ~]# chmod +x ./tcp_tsunami.ko
[root@California_VPS ~]# cp -rf ./tcp_tsunami.ko /lib/modules/$(uname -r)/kernel/net/ipv4
[root@California_VPS ~]# insmod tcp_tsunami.ko
[root@California_VPS ~]# depmod -a
[root@California_VPS ~]# echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
[root@California_VPS ~]# echo "net.ipv4.tcp_congestion_control=tsunami" >> /etc/sysctl.conf
```

没有报错都顺利完成的话，重启服务器。然后查看：  

```
[root@California_VPS ~]# lsmod | grep tsunami
tcp_tsunami            16384  5 
```

如果能看到 tcp_tsunami 证明 BBR 魔改版已安装成功。

## Debian/Ubuntu

### 1、开启 BBR

执行下面脚本一键开启  
 
```
wget --no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && chmod a+x BBR.sh && bash BBR.sh -f
```

### 2、安装魔改版BBR  

```
wget --no-check-certificate -qO 'BBR_POWERED.sh' 'https://moeclub.org/attachment/LinuxShell/BBR_POWERED.sh' && chmod a+x BBR_POWERED.sh && bash BBR_POWERED.sh
```

完成之后执行下面的命令检查是否安装成功  

```
lsmod |grep 'bbr_powered'
```

如果结果有 `bbr_powered` 则说明加载成功！

# 4、安装 SSR 
搬瓦工的后台控制面板（KiwiVM）已经提供了一键安装的功能，不过有些协议并不支持，如果想体验完整版的 SSR 的安装，还是自己动手吧！  
还是通过一键式傻瓜脚本安装： 

先下载脚本   

```
[root@California_VPS ~]# wget –no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
```

然后赋权、安装  

```
[root@California_VPS ~]# chmod +x shadowsocksR.sh 
[root@California_VPS ~]# ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

执行完安装命令会看到这个：  

```
#############################################################
# One click Install ShadowsocksR Server                     #
# Intro: https://shadowsocks.be/9.html                      #
# Author: Teddysun <i@teddysun.com>                         #
# Github: https://github.com/shadowsocksr/shadowsocksr      #
#############################################################

Please enter password for ShadowsocksR:
...
Please enter a port for ShadowsocksR [1-65535]:
...
Please select stream cipher for ShadowsocksR:
...
Please select protocol for ShadowsocksR:
...
Please select obfs for ShadowsocksR:
...
```

需要你选择相应的参数，前两个比较好理解，一个是SSR的密码，一个是端口；后面三个就比较复杂了，关于加密方法和协议介绍请看[wiki文档](https://github.com/gaoshilei/shadowsocks_install/blob/master/shadowsocksR-wiki/ShadowsocksR%20协议插件文档.md)  

如果不想看文档，可以使用推荐参数配置  
加密：`chacha20`和`aes-256-cfb8`
协议：`auth_chain_a`和`auth_aes128_md5`和`auth_aes128_sha1`
混淆：`plain`,`http_simple`,`http_post`,`tls1.2_ticket_auth`  


卸载 SSR：`./shadowsocksR.sh uninstall`

SSR 一些常用的命令  

```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
```

日志路径

```
配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocksr.log
代码安装目录：/usr/local/shadowsocks

```

如果后面想修改 SSR 的一些配置可以直接修改 `shadowsocks.json` ，然后重启 SSR 即可。  

最后配置 SSR 客户端就可以畅快的刷 YouTube 了，当然你也可以去 tumblr。【手动邪恶】    
**奉上各个平台 SSR 的下载链接：**  
MAC版下载地址：[ShadowsocksX-NG.1.5.1.zip](https://github.com/iMeiji/shadowsocks_install/releases/download/0.13/ShadowsocksX-NG.1.5.1.zip)  
windows版下载地址：[ssr-win.4.7.0-fix.7z](https://github.com/iMeiji/shadowsocks_install/releases/download/0.13/ssr-win.4.7.0-fix.7z)  
iOS版下载地址：[wingy](https://itunes.apple.com/us/app/wingy-http-s-socks5-proxy-utility/id1178584911?mt=8)（中国区已下架，换个美国账号下载）
安卓版下载地址：[ssr-android-3.4.0.5.apk](https://github.com/iMeiji/shadowsocks_install/releases/download/0.13/ssr-android-3.4.0.5.apk)

最后看看效果吧，YouTube 1080P 视频亲测截图  

![YouTube 1080P视频](http://oeat6c2zg.bkt.clouddn.com/QQ20171107-174706@2x.png)


再来一张YouTube的下载速度：  

![YouTube视频下载速度](http://oeat6c2zg.bkt.clouddn.com/QQ20171107-175234@2x.png)  

是不是感觉帮帮哒！
