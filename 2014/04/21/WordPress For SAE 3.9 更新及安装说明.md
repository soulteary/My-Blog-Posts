# WordPress For SAE 3.9 更新及安装说明

最近工作颇忙，抽时间把答应的跨版本升级做了一下，改动较多，稍后上传GIT，本次开始不提供WIKI记录，相信机智的你通过LOG日志也能知道是怎么回事 :D

除了官方的一些改进和新特点外，WP4SAE 3.9主要改进包括以下内容：

*   解决了安装上传安装包时会出现的一些问题。
*   解决了跨版本更新时会出现白屏的问题，以及备用解决工具。
*   完善了出错的一些提示。
*   去掉了大量的禁用函数，减少运行时出错的可能。
*   完善了发送邮件插件，增加了修改密码工具，以及用户级别自助查错的工具。
*   更多的细节，请自行体验 :D

使用WP4SAE的童鞋多数都是冲着免费简单而来，即使服务器偶尔挂掉，但是长期来看，相对稳定，所以在安装的问题上花了一些时间，做了一些优化，让大家在安装和升级的过程中更爽一点。

安装过程比较简单
 
**一键安装的同之前，如果在安装过程中出现问题，一概是官方服务器环境的问题（这个安装脚本用了至少2年没有问题）**

如果你通过一键安装失败，也不必忧桑，跟着下面的步骤一步一步来，很快就搞定你的WordPress 3.9了。

首先登录SAE官方网站，使用微博登录后，找到你的应用后台，选择“代码管理”，建议使用“创建一个版本”，新建一个版本进行安装。

[![1.create_version](https://attachment.soulteary.com/2014/04/21/1.create_version.png "1.create_version")](https://attachment.soulteary.com/2014/04/21/1.create_version.png)

这里为了演示，我创建了一个名为版本1的代码仓库，点击最前面的按钮，将默认版本设置为版本1，并将代码包上传至这个新建的代码仓库中。

[![2.switch_version](https://attachment.soulteary.com/2014/04/21/2.switch_version.png "2.switch_version")](https://attachment.soulteary.com/2014/04/21/2.switch_version.png)

代码上传完毕后，将这个弹出的框关闭掉。

[![3.upload_install_package](https://attachment.soulteary.com/2014/04/21/3.upload_install_package.png "3.upload_install_package")](https://attachment.soulteary.com/2014/04/21/3.upload_install_package.png)

打开你的博客地址，并在后面加上wp-admin，如图，访问博客后台的时候会提示升级数据库，点击按钮即可。

[![4.view_admin_panel](https://attachment.soulteary.com/2014/04/21/4.view_admin_panel.png "4.view_admin_panel")](https://attachment.soulteary.com/2014/04/21/4.view_admin_panel.png)

刚刚打开的SAE网站后台大家应该没有关闭吧，关闭了？再打开一次吧~ 在你的应用信息处点击查看按钮，将Secret Key 复制出来，等下要用。

[![5.copy_sk](https://attachment.soulteary.com/2014/04/21/5.copy_sk.png "5.copy_sk")](https://attachment.soulteary.com/2014/04/21/5.copy_sk.png)

访问你的博客地址，后面添加wp-tweak-tools.php，将刚刚复制的密码粘贴进来，点击验证。

[![6.tweak_tools](https://attachment.soulteary.com/2014/04/21/6.tweak_tools.png "6.tweak_tools")](https://attachment.soulteary.com/2014/04/21/6.tweak_tools.png)

稍等片刻，当页面结果显示一切正常的时候，你的博客就彻底升级完成了。

[![7.finall](https://attachment.soulteary.com/2014/04/21/7.finall.png "7.finall")](https://attachment.soulteary.com/2014/04/21/7.finall.png)

如果你和我一样，把SAE上的网站密码忘记了，可以使用这个小工具来进行帐号密码更新，同样，把刚刚复制的SAE应用SK粘贴到工具验证框即可，这里注意一下，修改密码可以填写任何你想更新的帐号，而并非仅限工具列出的管理员帐号:D

[![8.reset-password](https://attachment.soulteary.com/2014/04/21/8.reset-password.png "8.reset-password")](https://attachment.soulteary.com/2014/04/21/8.reset-password.png)

## FAQ

下面是一些FAQ，等发布后慢慢完善。


- [![1.error-sql](https://attachment.soulteary.com/2014/04/21/1.error-sql.png "1.error-sql")](https://attachment.soulteary.com/2014/04/21/1.error-sql.png)
  - 如果你出现了上面这个错误提示。 [MySQL坏了怎么办。](#MYSQL-ERROR) MySQL是WordPress使用的数据库，SAE上暂时没有提供程序自动初始化MySQL的接口（自动部署应用有该接口，但是对于已安装应用不可使用），所以我们需要手动自己开启。 别看描述这么多，其实操作很简单，登录SAE后台选择你的博客的应用，点击“初始化数据库”即可。 
  - [![1.error-sql-init](https://attachment.soulteary.com/2014/04/21/1.error-sql-init.png "1.error-sql-init")](https://attachment.soulteary.com/2014/04/21/1.error-sql-init.png)
- [![2.error-mc](https://attachment.soulteary.com/2014/04/21/2.error-mc.png "2.error-mc")](https://attachment.soulteary.com/2014/04/21/2.error-mc.png) 如果你遇到了上面的错误。
    - [MC坏了怎么办。](#MC-ERROR) MC是WordPress使用的内存缓存服务，有了它，WordPress就告别了慢吞吞的一般机械磁盘文件缓存，同样的，对于非自动部署应用，是不可以自动开启的，需要手动开启。 也是登录后台，选择你的应用，点击初始化，容量的话，如果你的博客访问量不大，选择1-2MB就可以了。
    - [![2.error-mc-init](https://attachment.soulteary.com/2014/04/21/2.error-mc-init.png "2.error-mc-init")](https://attachment.soulteary.com/2014/04/21/2.error-mc-init.png)
- [Storage坏了怎么办。](#STOR-GONE)

