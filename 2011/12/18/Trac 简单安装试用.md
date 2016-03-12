# Trac 简单安装试用

想要使用trac需要两个条件，一个是支持的平台，第二个是各种配置文件...
第一个条件是支持python，虽然蟒蛇目前到3.X了..但是如图蟒蛇官方和trac的提示,2.X是目前的版本,以及trac需要ver>2.4 && < 3.0的python..然后是下载win32的setuptools(version >= 0.6),Genshi(version >= 0.6),安装到python安装目录.

但是,这样还是太麻烦..因为假如你也只是想试一试trac.因为刚刚的事情还木有完.还要选择和安装数据库...
btw,不是所有人都可以apt-get install,然后按y解决问题..

so,找个现在流行的集成环境才是王道..as just have a try..
首先想到的是类似的xmapp的集成环境..省事就是硬道理...想到apache..想到 apache subversion,
然后看到他有一些win32编译版本...i like it...省事就是硬道理,again..

<!-- more -->

[visual svn](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.visualsvn.com%2Fserver%2Fdownload%2F&key=32063edfa1c9913108d6032e4552566e),小巧易用就选他了. 然后发现它可以集中管理这些周边软件.. 比如这篇文字...

[VISUALSVN SERVER // Installing Trac with VisualSVN Server](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.visualsvn.com%2Fserver%2Ftrac%2F&key=547c659e081d0b99e401cb02b367175a) 


首先下载 VisualSVN Server 2.5.2 或其他更高的版本. 安装并使用默认配置..你需要做的就是type enter and again. 在SVN SERVER的repository(库)里建立一个"**MyProject**"，如果使用subversion验证的话(安装的时候),建立一个可以使用的subversion用户。 

下载VisualSVN-Server-2.5.2.27089-Trac-0.12.2.zip 然后解压缩到_%VISUALSVN_SERVER%_,顺便提醒一下,因为后面很多变量都是建立在绝对路径C:\的。 

所以和测试symbian一样,老老实实的安装到C盘吧. **新建一个文件夹TRAC在C盘根目录**. 如果是登录用户不是administrator,给trac目录"完全控制"权限. 然后挨着在CMD里输入下面的执行..当然你也可以复制粘贴.. 如果你修改了一些安装路径,下面的命令行也注意修改..

```dos
"%VISUALSVN_SERVER%trac\trac-admin.bat" C:\Trac\MyProject initenv
```

输入path的时候,键入

```dos
C:\Repositories\MyProject
```

继续输入cmd命令,把库关联起来..

```dos
"%VISUALSVN_SERVER%trac\trac-admin.bat" c:\Trac\MyProject repository add MyProject C:\Repositories\MyProject svn
```

继续继续..

```dos
@"%VISUALSVN_SERVER%trac\trac-admin.bat" C:\Trac\MyProject changeset added "%1" "%2"
```

```dos
@"%VISUALSVN_SERVER%trac\trac-admin.bat" C:\Trac\MyProject changeset modified "%1" "%2"
```

添加系统变量.因为python的包需要python path..而这个绿色环境木有这个变量..btw,汉化trac也有类似小问题.. 后面跟着的变量是你的VisualSVN Server安装目录..你可以检查%VISUALSVN_SERVER%目录的位置,修改字符串.

```dos
PYTHONHOME=C:\Program Files\VisualSVN Server\trac\python
```

如果你使用apache subversion的验证,设置一下apache的配置文件.. 集成环境的位置是 _%VISUALSVN_SERVER%conf\httpd-custom.conf_

```text
LoadModule python_module "trac/python/mod_python_so.pyd"
LoadModule authz_user_module bin/mod_authz_user.so
```

最后说一下,visualsvn包含Apache Subversion 1.7.2...这个发布日期是..2011-12-05..激动~ 最后就是enjoy了..附送一个网站.. http://trac-hacks.org/

