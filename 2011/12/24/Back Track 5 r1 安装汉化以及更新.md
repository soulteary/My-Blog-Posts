# Back Track 5 r1 安装汉化以及更新

[![Back Track 5 中文](https://attachment.soulteary.com/2011/12/24/Screenshot.jpg "Back Track 5 中文")](https://attachment.soulteary.com/2011/12/24/Screenshot.jpg) 

## 写在前面:

> 昨天晚上切换到ubuntu玩，实在不能接受11.10到速度了，而且实在怀念GNOME 2，本来打算效仿论坛虾米去降级的。 但是搜索了一下，似乎GNOME 3无法降级，或者说太麻烦了。于是打算重新安装ubuntu 8～10，下载DVD镜像的时候，突然发现，移动硬盘里存着Back Track R1 的 ISO镜像，蛋疼的事情<span style="color: #ffcc00;">开始了</span>。


## Back Track是什么

Back Track 简单的说是一个基于linux流行发行版的发行版...集成了许多经典的安全审计工具。当然刀是双刃的，你可以用来保护别人，也可以用来伤害他人，希望用于后者的少一些。更多资料百度一下。</pre>

## 安装Back Track 5

这个很EASY，推荐制作LIVE USB进行安装。 具体的可以看[官方页面](http://www.ubuntu.com/download/ubuntu/download "UBUNTU 安装方法")。 选择到时候提示一下，Back Track 5 R1 是UBUNTU 10.04，工具有Back Track 5 R1就选它，木有就选择UBUNTU 10.04 安装过程中推荐联网，似乎网线插着的话，Back Track 5（下面都简称BT5）安装速度会快一些，(感觉旧的UBUNTU安装过程中用DO LOOP来等待网络反馈)，一路NEXT设置好你的键盘布局，时区等信息后，耐心等待它安装完成，提示重启，点击按钮，然后等待计算机完全重启到时候拔掉U盘。(因为LIVE USB结束所有读写是在下一次机器引导之前，不想你的U盘有闪失的话) 


## 启动Back Track 5 R1

BURG熟悉的界面出现在眼前后，选择BT5，回车等待系统启动。大约半分钟，提示输入帐号和密码。 使用默认帐号 **<span style="color: #ff0000;">root</span>**，密码**<span style="color: #ff0000;">toor</span>**，然后回车，这里提示一下，linux的密码输入为了安全是不显示输入的，输入好了，回车就好。 你看不到*型号或者你打的字符不代表您木有打上字，某舍友可爱的说（你没有打上字...我汗） 看到提示你的密码是默认到toor和输入startx进入图形化界面的时候，安装总算完成了。 不用犹豫，键入**<span style="color: #ff0000;">startx</span>**，进入BT5. **开始汉化** CTRL+ALT+T打开终端，直接输入 **lsb_release -a** ,确认你的ubuntu 版本，可以看到10.04，LTS. UBUNTU 10.04是长期支持版本,所以先把Back Track 5 R1精简掉的一些东西找回来吧。 如果想升级ubuntu 或者美化这些的，不着急，慢慢来，心急吃不了热豆腐。 **顺便提醒一下，虽然Back Track 5 R1的软件都是编译好的,但是一味升级UBUNTU版本,使其他的一些环境(库,依赖升级) 或许会使某些程序无法正常运行,所以说,适合的才是最好的,UBUNTU的长期支持版本也是这个道理,WINDOWS XP经久不衰也是如此.** 打开**[Official Archive Mirrors for Ubuntu](https://launchpad.net/ubuntu/+archivemirrors "官方镜像站点")**， 查找你觉得网速不会坑你的源，我选择的是可爱的青岛大学。原因是第一它到带宽最大，第二它更新是最新的那批。 虽然UBUNTU 社区的例子告诉我们修改文件要用nano，但是实践告诉我们，省事的方法应该这么做

> **gedit /etc/apt/sources.list**

把下面的内容复制，然后粘贴进gedit，然后保存，关闭gedit。下面的是青岛大学的源地址。 如果的连接速度不理想，访问上面的官方镜像站点，用你连接速度快的源替换掉下面的内容。

> deb http://mirror.osqdu.org/ubuntu/ YOUR_UBUNTU_VERSION_HERE main deb-src http://mirror.osqdu.org/ubuntu/ YOUR_UBUNTU_VERSION_HERE main

更新源列表中的包数据，准备开始补全我们的BT5.

> apt-get update

接着开始下载中文数据和ubuntu中的经典工具，比如新立得，抓图，远程管理，桌面分享，当然还有语言支持。 这里开始是开着另外一台机器，挨着敲名字，然后发现很蛋疼，我既懒且笨，所以，百度了到了一篇名称较全的日志。 感谢作者了：[http://hi.baidu.com/adomore/blog/item/f1b831a3e5388e9746106464.html](http://promiseforever.com/redirect?url=http%3A%2F%2Fhi.baidu.com%2Fadomore%2Fblog%2Fitem%2Ff1b831a3e5388e9746106464.html&key=745b1ab1113afb17a7d7bbf8ddab8fd2 "虚拟机汉化Back Track 5 R1") 但是他似乎是在虚拟机里做到哦，而且应该自己之前有apt-get install 其他的东西了，所以指令要修改下。 首先先把大体工具和资料都更新下来，特别喜欢linux的安装依赖，会帮你把使用到的库都一起下载，不会出现缺少文件的情况， 如果你仅仅需要中文支持

> apt-get install language-pack-gnome-zh language-support-zh language-pack-gnome-zh-base language-pack-zh language-pack-zh-base language-selector gnome-utils

如果你也觉得UBUNTU的软件更新,远程桌面,以及软件中心,新立得好用的话,可以这么下:

> **apt-get install synaptic language-pack-gnome-zh language-support-zh language-pack-gnome-zh-base gnome-system-tools language-pack-zh language-pack-zh-base language-selector gnome-utils tsclient vinagre update-manager software-center**

漫长等待，更新完毕后执行下面的指令

> **apt-get install language-selector update-manager**

是不是报错了，恩，肯定的，BT5 自己带着的库版本比软件依赖到高。那么怎么办捏。卸载掉高版本的两个库，指令是:

> **apt-get remove language-selector-common apt-get remove update-manager-core**

然后再执行一次

> **apt-get install language-selector update-manager**

接着修改环境变量

> **gedit /etc/default/locale**

> **LANG="zh_CN.UTF-8" LANGUAGE="zh_CN:zh"**

然后重启,你的中文版Back Track 5 R1就搞定了.接下来,如果你想要升级的话,只需要把刚刚的源修改为你想升级的版本的源, 比如你想升级Back Track 5 R1 为UBUNTU 10.10,只需要修改源为

> deb http://mirror.osqdu.org/ubuntu/ maverick main deb-src http://mirror.osqdu.org/ubuntu/ maverick main

然后执行

> **apt-get update && apt-get -y upgrade**

补充一下UBUNTU 11.04的源

> deb http://mirror.osqdu.org/ubuntu/ natty main deb-src http://mirror.osqdu.org/ubuntu/ natty main

UBUNTU 11.10的源

> deb http://mirror.osqdu.org/ubuntu/ oneiric main deb-src http://mirror.osqdu.org/ubuntu/ oneiric main

最后补充一点,http://forum.ubuntu.org.cn/viewtopic.php?t=82664

