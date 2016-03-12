# 配置WIN主机的一些小细节

记录一下今天看到的一些小事情. 由于一些比较特殊的需求,决定把用了6个月的VPS换成windows主机, 于是备份了centos的一些软件配置, 以及data目录下的文件. 首先把系统换成了windows 2K3, 为了图省事, 直接使用了[xampp](http://www.apachefriends.org/zh_cn/xampp-windows.html), 因为看到了新的**XAMPP 1.8.3**中包含了**PHP 5.5.3**和**MySQL 5.6.11**,于是果断下载使用,但是发现不能使用...

网上搜索了一下,看到了这个[release-log](https://github.com/php/php-src/blob/php-5.5.0alpha1/NEWS),中**"Drop Windows XP and 2003 support."** 毫不犹豫的把windows 2k3 换成了 windows 2k8, 毫无疑问的可以使用...

在考虑传统服务器端使用什么软件的时候,想了一下, apache+iis(共享80)/ apache/ iis/...

省事的话,iis 就好了,不过因为熟悉程度还是选择了使用apache, 选择的过程中, 发现了一个看起来不错的东西 "[WinCache Extension for PHP](http://www.iis.net/downloads/microsoft/wincache-extension)", 以及 "[PHP Manager for IIS](http://phpmanager.codeplex.com/)" 话说这里有个额外的扩展信息,就是选择php的版本,当然可以直接使用xampp内置的版本 ^ ^ 官方给我们的[下载页面](http://windows.php.net/download/)有4个选项: VC11 x86 Non Thread Safe/ VC11 x86 Thread Safe/ VC11 x64 Non Thread Safe/ VC11 x64 Thread Safe, x86/x64区别大家都知道, Non Thread Safe/Thread Safe到底该选择那一种呢? 好在官方在页面左侧给了我们答案:

*   如果你使用IIS的话,那么 应该使用Non-Thread Safe (NTS)版本的PHP
*   如果你使用早期从apache.org官方下载编译的apache 1/2的话,你需要使用VC6(Visual Studio 6)编译的版本, 请不要使用VC9+的PHP版本
*   VC9或VC11 (Visual Studio 2008/ 2012编译的版本), 如果是VC9的话,需要安装 Visual C++ Redistributable for Visual Studio 2008/2012 SP1 x86 or x64

不过似乎看的不过瘾,搜索了一下,发现百度文库里有一些其他的信息(待考):

*   先从字面意思上理解，Thread Safe是线程安全，执行时会进行线程（Thread）安全检查，以防止有新要求就启动新线程的CGI执行方式而耗尽系统资源。Non Thread Safe是非线程安全，在执行时不进行线程（Thread）安全检查。
*   PHP的两种执行方式：ISAPI和FastCGI。ISAPI执行方式是以DLL动态库的形式使用，可以在被用户请求后执行，在处理完一个用户请求后不会马上消失，所以需要进行线 程安全检查，这样来提高程序的执行效率，所以如果是以ISAPI来执行PHP，建议选择Thread Safe版本；而FastCGI执行方式是以单一线程来执行操作，所以不需要进行线程的安全检查，除去线程安全检查的防护反而可以提高执行效 率，所以，如果是以FastCGI来执行PHP，建议选择Non Thread Safe版本。
*   选择以下这些版本需要注意的是MYSQL在2008R2下可以选择64位的,PHP的VC9是针对IIS的,VC6针对apache的,线程安全和非安全版本本次选择的是线程安全版本, PHP线程安全版本无法加载wincache,所以我们用Xcache作为替代,如果想用wincache就选用非线程安全版本

然后需要实现的是ssh, 这货在centos中是默认有的, 那么windows怎么破呢, 有人已经为我们搞定了一切:[CopSSH](https://www.itefix.no/i2/copssh), Free Edition（3.1.4） 免费版本就可以使用, 如果你非要使用GUI版本的话，可以网上找找，不过这种软件建议使用官方而非hacked软件，减少风险。当然如果你想自己动手全部体验一下：Cygwin + OpenSSH 也不错。 软件的安装很简单，一路next即可，有一个提示内置账户和密码的界面也可以一路next，因为和提示中说明的一样，你或许一辈子都不会使用到这个帐号，如果你非要更改的话，请参考说明，不要使用administrator帐号等。 安装之后，先在系统中创建一个新的帐号，用于ssh登录，然后打开安装好的软件，选择该用户，继续一路next即可。 不过这个里面的linux工具集有点少（可以连接就不错了诶！），如果想用的顺手，自己添加诸如“alias ll='ls -lht'”... 输入**mount**可以查看windows下的模拟的分区挂载情况：

```bash
$ mount
D:/Program Files (x86)/ICW/etc/terminfo on /usr/share/terminfo type ntfs (binary,noacl)
D:/Program Files (x86)/ICW/bin on /usr/bin type ntfs (binary,noacl)
D:/Program Files (x86)/ICW/lib on /usr/lib type ntfs (binary,noacl)
D:/Program Files (x86)/ICW on / type ntfs (binary,noacl)
C: on /cygdrive/c type ntfs (binary,noacl,posix=0,user,noumount,auto)
D: on /cygdrive/d type ntfs (binary,noacl,posix=0,user,noumount,auto)
```

如果你要修改端口号，不管是mstsc或者ssh连接服务器修改**/etc/sshd_config**即可。（基本和centos上操作类似） 不过实际测试发现，ssh断开后，windows主机的进程并没有结束...(任务管理器中残留了好几个以创建的用户名启动的bash *32进程) 计划在一个web rest接口，来干掉这些僵尸进程。 软件仓库的话，明天继续...

