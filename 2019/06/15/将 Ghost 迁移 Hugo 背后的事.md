# 将 Ghost 迁移 Hugo 背后的事

今天打开 Ulysses 看到官方说 v16 版本支持直接发布文章到 Ghost 博客程序，想起了上个月迁移 Ghost 程序的事情。如果你在犹豫是否要弃用或暂时弃用它，本篇文章或许可以作为一个参考。

大概二十天前的一个周末，我把还在使用 Ghost 程序的两个网站全部使用 Hugo 替换掉了，看似简单的操作，背后是五年的“等待”。

没错，这套代码在线上跑了五年多，相比我使用的其他程序的网站的“持续”时间段都长许多，既然使用了这么久，那么为什么要替换呢？

## 为何迁移

Ghost 的发展并没有原计划中的顺利，不论是从平台人数、还是从功能迭代速度来说。

每当 Ghost 有新版本的时候，我都会第一时间用容器跑个临时 demo 体验一下，但每次几乎都是“乘兴而来，扫兴而去”。

- 编辑器越来越不好用，资源管理也迟迟不启动
- 本地化全靠粉丝，并且几乎只有语言包（仅主题）这一块内容
- 插件市场进展缓慢

这三点几乎触及了长时间使用的用户的底线。

### 书写体验、文章管理体验差

非英文输入会出现的 Bug，从 17 年开始就有用户不断的上报，并且 ping 到了官方和编辑器的维护者：

- [https://github.com/TryGhost/Ghost/issues/9801](https://github.com/TryGhost/Ghost/issues/9801) 
- [https://github.com/bustle/mobiledoc-kit/issues/583](https://github.com/bustle/mobiledoc-kit/issues/583)

用户们甚至给了详细的解决方案，但是官方都迟迟未开始动作解决问题，而且可气的是，官方始终认为这个锅我们不背，**作为开发者，你的产品引用的依赖不是你的产品的一部分么**。

年初的时候，忍不住发了一个 `@bugfix` 的 npm 包，[https://github.com/soulteary/mobiledoc-kit](https://github.com/soulteary/mobiledoc-kit)，也建议了官方采取类似动作，先把这个旧账还上，然而等来的只有机器人的回复。

> This issue has been automatically marked as stale because it has not had recent activity. It will be closed if no further activity occurs. Thank you for your contributions.

然而用户的眼睛是雪亮的，这条回复上长出来十几个赞。

![就只需要一个简单动作，就能解决的 Bug](https://attachment.soulteary.com/2019/06/15/ime-bugs.png)

附件管理只支持图片，不提管理功能，单说支持文件类型就很忧伤。什么视频、压缩包，一律不支持。

另外，时至今日，还是不支持 CDN 功能，其实哪怕你支持回源模式的 CDN 都好呀。

这类功能，从回源模式到上传中间件，大家其实提过不少方案，比如我下面这个 2015 年的提交。

- [https://github.com/soulteary/Ghost/commits/master](https://github.com/soulteary/Ghost/commits/master)

### 本地化措施

虽然看起来相比第一条，这条没有那么重要，但是经年累月的使用，这个点不容忽视，举例来说。

中英文摘要处理是一样的嘛？

![中文的阶段策略真的是无语](https://attachment.soulteary.com/2019/06/15/content-truncated.png)

解决方法很简单，可以参考我上面提到的 2015 年的提交记录。

你想往文章中插入B站等中文站点的视频？官方支持列表可是这样。

![官方默认支持的文章集成工具](https://attachment.soulteary.com/2019/06/15/default-embed.png)

直接尝试嵌入，会得到下面的结果。

![测试嵌入视频](https://attachment.soulteary.com/2019/06/15/embed-res.png)

到官方市场看看，一堆英文友好的工具产品。

![官方市场的集成工具](https://attachment.soulteary.com/2019/06/15/integrations.png)

解决方法不是没有，只是要做很多 hacks ，这还只是视频，其他的文件类型就更不提了，而去因为第一点中提到的没有文件管理的概念，文章多了之后，除了自己写脚本批量处理文件，别无他法。

至于中文语言包，官方在拖了许久、明拒婉拒许多贡献者 PR 后，推出了网站主题支持语言汉化文件的措施…全界面汉化对于推广应用真的没有价值，还是你们的产品定位就只有工程师和官宣的主要的英文用户们？

** 作为开源项目，如果你不想让别人做某件事，为什么不在贡献文档中显著的声明一下呢？**

### 插件功能进展缓慢

几个月前，终于迎来了插件支持。

但是没有出现任何 `custom` 的新类型支持，那么封装这些插件的意义是什么，和几年前就可以通过编写模版 helper 来解决的不都是同一类问题嘛？

至于市场内容，上面一节的图片里有描述，几乎完全没有考虑中文用户。

## 迁移过程

如果你使用的是 SQLite 作为数据库，过程很简单，只需要导出文章数据，然后将资源也做一个简单迁移。

如果你忘记了密码，可以用参考下面的命令重置你的程序密码为 `password`。

```bash
# 官方容器镜像缺失 sqlite3 工具
apt update
apt install sqlite3 -y

# 找到你的数据库文件后打开它
sqlite3 ghsot.db

# 获取所有用户邮箱
select email from users;

# 比如这个就是你要修改的用户
# abc@soulteary.com

UPDATE users SET password='$2a$10$BQToDNdBtBKCvnrTmMi5m.NK.7i6Qx7YASs.jTkE86I5zqxzE8klC' WHERE email = 'abc@soulteary.com'

# 关闭数据库
.exit
```

## 其他

虽然我已经不再使用 Ghost 作为线上使用的网站程序，并且对它吐槽不断，但是或许有一天我依旧会使用它。

希望到时候，重启它的原因不光是因为 Node 技术栈，更多的是因为它在细节交互上更加用心、对用中文用户的态度有所转变。

—EOF