# 服务端升级 Ubuntu 20.04 LTS 记录

本文将介绍如何在当前时间点，将服务器版本的 Ubuntu 18.04 LTS 升级为最新的 Ubuntu 20.04 LTS，以及升级过程中的一些细节，希望能帮到跃跃欲试的 Ubuntu 同好。

将数据进行备份等操作需要自行处理，另外确保网络稳定，建议都在服务器跳板机上进行操作，更为稳妥。

当前这篇内容已经运行在 Ubuntu 20.04 LTS 系统环境中，:)

## 准备工作

先使用 `apt update` 看看有哪些内容可以升级。

```bash
apt update 

Hit:1 http://mirrors.aliyun.com/ubuntu bionic InRelease
Get:2 http://mirrors.aliyun.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:3 http://mirrors.aliyun.com/ubuntu bionic-security InRelease [88.7 kB]
Get:4 http://mirrors.aliyun.com/ubuntu bionic-updates/universe Sources [281 kB]                     
Get:5 http://mirrors.aliyun.com/ubuntu bionic-updates/main Sources [315 kB]                                               
Get:6 http://mirrors.aliyun.com/ubuntu bionic-updates/main amd64 Packages [915 kB]                                           
Get:7 http://mirrors.aliyun.com/ubuntu bionic-updates/main i386 Packages [669 kB]                                            
Get:8 http://mirrors.aliyun.com/ubuntu bionic-updates/main Translation-en [315 kB]                                           
Get:9 http://mirrors.aliyun.com/ubuntu bionic-updates/universe i386 Packages [1,014 kB]                                      
Get:10 http://mirrors.aliyun.com/ubuntu bionic-updates/universe amd64 Packages [1,065 kB]                                    
Get:11 http://mirrors.aliyun.com/ubuntu bionic-updates/universe Translation-en [331 kB]                                      
Hit:12 https://download.docker.com/linux/ubuntu bionic InRelease                     
Get:13 http://mirrors.aliyun.com/ubuntu bionic-security/universe Sources [168 kB]
Get:14 http://mirrors.aliyun.com/ubuntu bionic-security/main Sources [146 kB]        
Get:15 http://mirrors.aliyun.com/ubuntu bionic-security/main amd64 Packages [692 kB]
Get:16 http://mirrors.aliyun.com/ubuntu bionic-security/main i386 Packages [459 kB]  
Get:17 http://mirrors.aliyun.com/ubuntu bionic-security/universe amd64 Packages [657 kB]
Get:18 http://mirrors.aliyun.com/ubuntu bionic-security/universe i386 Packages [618 kB]
Fetched 7,823 kB in 3s (3,106 kB/s)                                                  
Reading package lists... Done
Building dependency tree       
Reading state information... Done
19 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

接着执行 `apt upgrade -y` 耐心等待软件升级完毕即可。如果你最近已经更新过，则会得到类似下面的内容提示。

```bash
apt update 

Reading package lists... Done                        
Building dependency tree       
Reading state information... Done
All packages are up to date.
```

如果你觉得升级过程中软件源比较慢，可以尝试替换源，比如像下面这样操作。

```bash
sed -i -e "s/mirrors.cloud.aliyuncs.com/mirrors.tuna.tsinghua.edu.cn/" /etc/apt/sources.list
```

## 升级过程的小麻烦们

当我们执行 `do-release-upgrade` 尝试进行升级的时候，可能会出现三种情况告诉我们不能够升级。

### 系统中还有未完全升级的软件

当你执行完毕命令后，可能会得到“Please install all available updates for your release before upgrading”的提示，这说明你其实并未将所有软件都完成升级。

```bash
do-release-upgrade

Checking for a new Ubuntu release
Please install all available updates for your release before upgrading.
```

你可能会好奇，我明明执行过 `update` 和 `upgrade` 了，为什么还会出现这种情况呢？

这里有一个很大的可能是，使用过 apt-mark 将部分软件版本锁定，需要先执行解锁操作，比如：

```bash
apt-mark unhold docker-ce
```

至于如何看到需要升级或者解锁的软件呢？

可以使用 `apt update && apt list --upgradable` 命令进行查询：

```bash
apt list --upgradable 

...
Reading package lists... Done
Building dependency tree       
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.

...

Listing... Done
docker-ce/bionic 5:19.03.8~3-0~ubuntu-bionic amd64 [upgradable from: 5:19.03.6~3-0~ubuntu-bionic]
N: There are 23 additional versions. Please use the '-a' switch to see them.
```

然后在再次执行 `apt upgrade -y` 即可完成所有软件升级。

但是千万别高兴太早，因为你还可能遇到接下来的事情。

### 解除系统升级锁定

当所有软件都升级就绪后，继续使用 `do-release-upgrade` 升级软件，会看到类似下面的提示。

```bash
do-release-upgrade 

Checking for a new Ubuntu release
There is no development version of an LTS available.
To upgrade to the latest non-LTS develoment release 
set Prompt=normal in /etc/update-manager/release-upgrades.
```

这里因为官网尚未正式打开版本推送，所以如果要想得到版本更新，需要对 `do-release-upgrade` 添加命令行参数 `-d`，来允许获取最新的升级包。

```TeXT
Usage: do-release-upgrade [options]

Options:

  -d, --devel-release   If using the latest supported release, upgrade to the
                        development release
```

### 作出升级路线选择

如果你是 Ubuntu 18.04 LTS 用户的话，此刻我们需要做出一个决定，是一个版本一个版本升级，还是直接跨版本升级，而如果是 Ubuntu 19.10 的用户则简单的多，因为不涉及跨版本问题，逐版本升级后半部分内容即可。

我们详细说说两种升级方式。

## Ubuntu 18.04 逐版本升级 Ubuntu 20.04

有一句流传甚远的话叫做“步子不能迈的太大”，某些时候也可以用在软件升级这件事上。

打开 `/etc/update-manager/release-upgrades` 文件，我们可以看到文件说明：

```bash
[DEFAULT]
# Default prompting behavior, valid options:
#
#  never  - Never check for, or allow upgrading to, a new release.
#  normal - Check to see if a new release is available.  If more than one new
#           release is found, the release upgrader will attempt to upgrade to
#           the supported release that immediately succeeds the
#           currently-running release.
#  lts    - Check to see if a new LTS release is available.  The upgrader
#           will attempt to upgrade to the first LTS release available after
#           the currently-running one.  Note that if this option is used and
#           the currently-running release is not itself an LTS release the
#           upgrader will assume prompt was meant to be normal.
Prompt=lts
```

将 `Prompt=lts` 修改为 `Prompt=normal` ，然后执行 `do-release-upgrade -d`，会开始第一阶段升级：

```bash
do-release-upgrade -d


Checking for a new Ubuntu release
Get:1 Upgrade tool signature [1,554 B]                                                                                       
Get:2 Upgrade tool [1,329 kB]                                                                                                
Fetched 1,331 kB in 0s (0 B/s)                                                                                               
authenticate 'eoan.tar.gz' against 'eoan.tar.gz.gpg' 
extracting 'eoan.tar.gz'

Reading cache
...
```

根据实际情况，我们“一路 Next”之后，在即将完成升级时，会看到下面的提示内容：

```bash
。。。
System upgrade is complete.

Restart required 

To finish the upgrade, a restart is required. 
If you select 'y' the system will be restarted. 
```

待系统重启之后，登陆系统会看到系统已经成功升级为 Ubuntu 19.10：

```TeXT
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Ubuntu 20.04 LTS is out, raising the bar on performance, security,
   and optimisation for Intel, AMD, Nvidia, ARM64 and Z15 as well as
   AWS, Azure and Google Cloud.

     https://ubuntu.com/blog/ubuntu-20-04-lts-arrives
```

我们将 `/etc/update-manager/release-upgrades` 文件中数值修改为 `Prompt=lts`  ，再次执行 `do-release-upgrade -d` 就可以开始第二阶段的升级了，操作过程和上面没有区别，一杯水的功夫，服务端再次重启，Ubuntu 20.04 LTS 就升级完毕啦。

```TeXT
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-26-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Ubuntu 20.04 LTS is out, raising the bar on performance, security,
   and optimisation for Intel, AMD, Nvidia, ARM64 and Z15 as well as
   AWS, Azure and Google Cloud.

     https://ubuntu.com/blog/ubuntu-20-04-lts-arrives
```

验证完毕逐版本升级，我们试试一步到位的跨版本升级。

## Ubuntu 18.04 跨版本升级 Ubuntu 20.04

跨版本升级相当于是逐版本升级的“偷懒版”，偷懒当然要到极致，升级软件包也可以使用 `apt full-upgrade -y` 这个命令来做。

和逐版本升级不同的是，我们不再需要修改 `release-upgrades` 配置文件，在升级前只需要确认 `/etc/update-manager/release-upgrades` 文件的数值是否设置为 `lts` 即可。

确认数值正确后，执行 `do-release-upgrade -d` ，根据自己需求进行升级配置选择即可，“一路 Next” 之后，Ubuntu 20.04 就升级完毕啦。

## 最后

距离将所有机器升级到 18.04 [刚巧一年](https://soulteary.com/2019/04/06/configure-ubuntu-18-04.html)， Ubuntu 20.04 LTS 的到来，算是一个惊喜。

一般情况下，我们使用 `update`, `upgrade`, `do-release-upgrade` 组合技应该就能顺畅完成升级，但是在当前时间点，官网还未完全正式提供 release 升级方案，所以就有了这篇文章。

呃，促成这篇文章还有一个原因，回家后直接睡觉忘记喂猫，被毛孩子抗议叫醒...

--EOF