# CentOS 基础操作

chrome安装了一个ssh,骤然对linux兴趣扩大无数倍,或许是因为是在浏览器里敲代码,隔阂感为零了嘛? 挨着敲了一下,找了一下手感,话说node跑循环感觉好逆天-,- yum install了一个node,100w的循环,没多久就完了. 首先是查看机器的基础信息

```bash
#查看CPU
[root@centos bin]# more /proc/cpuinfo | grep "model name"
model name      : QEMU Virtual CPU version 0.13.0

[root@centos bin]# grep "model name" /proc/cpuinfo
model name      : QEMU Virtual CPU version 0.13.0

[root@centos bin]# grep "model name" /proc/cpuinfo | cut -f2 -d:
 QEMU Virtual CPU version 0.13.0

#查看内存
[root@centos bin]# grep MemTotal /proc/meminfo
MemTotal:         506124 kB

[root@centos bin]# grep MemTotal /proc/meminfo | cut -f2 -d:
         506124 kB

[root@centos bin]# free -m |grep "Mem" | awk '{print $2}'
494

#查看CPU位数
[root@centos bin]# getconf LONG_BIT
64

[root@centos bin]# echo $HOSTTYPE
x86_64

[root@centos bin]# uname -a 
Linux centos 2.6.35.8-guest-64 #5 SMP Fri Apr 29 07:56:20 CST 2011 x86_64 x86_64 x86_64 GNU/Linux

#查看操作系统版本
[root@centos bin]# more /etc/redhat-release
CentOS release 5.4 (Final)

[root@centos bin]# cat /etc/redhat-release
CentOS release 5.4 (Final)

#查看内核版本
[root@centos bin]# uname -r
2.6.35.8-guest-64

[root@centos bin]# uname -a
Linux centos 2.6.35.8-guest-64 #5 SMP Fri Apr 29 07:56:20 CST 2011 x86_64 x86_64 x86_64 GNU/Linux

#查看时间
[root@centos bin]# date
Thu Jan 17 23:51:32 CST 2013

#查看系统分区
[root@centos bin]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/vda1             7.8G  7.1G  309M  96% /
tmpfs                 248M     0  248M   0% /dev/shm
/dev/vdb               20G  1.1G   18G   6% /data1
/dev/vdb               20G  1.1G   18G   6% /data2

#查看更具体的磁盘状况
[root@centos bin]# fdisk -l

Disk /dev/vda: 8598 MB, 8598323200 bytes
255 heads, 63 sectors/track, 1045 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           1        1045     8393931   83  Linux

Disk /dev/vdb: 21.4 GB, 21474836480 bytes
16 heads, 63 sectors/track, 41610 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes

Disk /dev/vdb doesn't contain a valid partition table

Disk /dev/vdc: 1073 MB, 1073741824 bytes
16 heads, 63 sectors/track, 2080 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes

Disk /dev/vdc doesn't contain a valid partition table
[root@centos bin]# 

#当前目录下文件夹大小
[root@centos bin]# du -sh
21M     .

#指定目录下的文件大小
[root@centos bin]# du /etc -sh
97M     /etc

#查看安装软件的列表
[root@centos bin]# cat -n /root/install.log
     1  Installing libgcc - 4.1.2-46.el5.x86_64
     2  warning: libgcc-4.1.2-46.el5: Header V3 DSA signature: NOKEY, key ID e8562897
     3  Installing libgcc - 4.1.2-46.el5.i386
     4  Installing setup - 2.5.58-7.el5.noarch
     5  Installing filesystem - 2.4.0-2.el5.centos.x86_64
     6  Installing basesystem - 8.0-5.1.1.el5.centos.noarch
     7  Installing centos-release-notes - 5.4-4.x86_64
     8  Installing nash - 5.1.19.6-54.x86_64
     9  Installing gnome-mime-data - 2.4.2-3.1.x86_64
    10  Installing cracklib-dicts - 2.8.9-3.3.x86_64
#安装软件的数量
[root@centos bin]# more /root/install.log | wc -l
820

#查看已经安装的软件包
[root@centos bin]# rpm -qa
gnome-mime-data-2.4.2-3.1
bzip2-libs-1.0.3-4.el5_2
sed-4.1.5-5.fc6
elfutils-libelf-0.137-3.el5
binutils-2.17.50.0.6-12.el5

#查看已经安装的软件包个数
[root@centos bin]# rpm -qa | wc -l
902

#查看已经安装的软件包个数
[root@centos bin]# yum list installed | wc -l
900

#查看键盘布局
[root@centos bin]# cat /etc/sysconfig/keyboard
KEYBOARDTYPE="pc"
KEYTABLE="us"

#查看键盘布局
[root@centos bin]# cat /etc/sysconfig/keyboard | grep KEYTABLE | cut -f2 -d=
"us"

#查看selinux
[root@centos bin]# sestatus
SELinux status:                 disabled

#查看selinux
[root@centos bin]# cat /etc/sysconfig/selinux
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - SELinux is fully disabled.
SELINUX=disabled
# SELINUXTYPE= type of policy in use. Possible values are:
#       targeted - Only targeted network daemons are protected.
#       strict - Full SELinux protection.
SELINUXTYPE=targeted

# SETLOCALDEFS= Check local definition changes
SETLOCALDEFS=0 
[root@centos bin]# 

#网卡信息
[root@centos ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:XX:XX:XX:XX:XX 
          inet addr:XX.XXX.XXX.XXX  Bcast:XX.XXX.XXX.XXX  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:26364068 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8390991 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:13939243718 (12.9 GiB)  TX bytes:4236593666 (3.9 GiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:154781 errors:0 dropped:0 overruns:0 frame:0
          TX packets:154781 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:31117569 (29.6 MiB)  TX bytes:31117569 (29.6 MiB)

[root@centos ~]# 

#查看IP
[root@centos ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 | grep IPADDR
IPADDR=XX.XXX.XXX.XXX
#查看IP
[root@centos ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0 | grep IPADDR | cut -f2 -d=
XX.XXX.XXX.XXX
#查看IP
[root@centos ~]# ifconfig eth0 |grep "inet addr:" |awk '{print $2}'|cut -c 6-
XX.XXX.XXX.XXX
#查看IP
[root@centos ~]# ifconfig   | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk '{ print $1}'
XX.XXX.XXX.XXX

#查看网卡
[root@centos ~]# cat /etc/sysconfig/network
NETWORKING=yes
NETWORKING_IPV6=no
HOSTNAME=centos

#DNS配置
[root@centos ~]# cat /etc/resolv.conf
; generated by /sbin/dhclient-script
nameserver 8.8.8.8
nameserver XX.XXX.XX.XX
search localdomain

#默认语言
[root@centos ~]# echo $LANG $LANGUAGE
en_US.UTF-8

#默认语言
[root@centos ~]# cat /etc/sysconfig/i18n
LANG="en_US.UTF-8"
SYSFONT="latarcyrheb-sun16"

#查看时区设置
[root@centos ~]# cat /etc/sysconfig/clock
ZONE="Asia/Shanghai"
UTC=false
ARC=false

#查看主机名称
[root@centos ~]# hostname
centos

查看主机名称和网络配置
[root@centos ~]# cat /etc/sysconfig/network
NETWORKING=yes
NETWORKING_IPV6=no
HOSTNAME=centos
```
