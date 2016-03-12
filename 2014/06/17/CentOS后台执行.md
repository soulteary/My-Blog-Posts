# CentOS后台执行

基础知识，偷懒记录一下，用的时候翻看好了。

centos 有蛮多可以后台执行的辅助命令，诸如：fg；bg；jobs；&；nohup；ctrl+z。

甚至你可以安装类似screen或者替代品tmux等等。

不过如果只是为了运行一个任务到后台且保持断开ssh后任务不断，且不想再搭理这个程序（如果它可以一直正常运行），那么使用下面的命令可以很轻松完成：

```bash
nohup 你的命令 > /dev/null 2>&1 &
```

如果你说万一重启了怎么办，或者进程挂了怎么办，那么使用定时任务好了：

```bash
#!/bin/bash
if ps -ef | grep "你执行的命令的执行文件的关键词，可以被搜索到以及被关闭" | grep -v "grep" then;
    echo "everything is ok."
else
    echo "try to restart tools."
    你的命令 > /dev/null 2>&1 &
fi
```

忘记补crontab了， 先编辑crontab：

```bash
crontab -e
```

然后比如要1分钟检查一次：

```text
*/1 * * * * /你的脚本的位置
```

最后检查crontab，

```bash
tail -f /var/log/cron
```

