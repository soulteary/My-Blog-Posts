# CENTOS下网站数据备份

最近对centos非常感兴趣，出于备份文件的需要，写了一个脚本，记录一下过程，以免以后用的少，忘记了。

首先，要打包的目录中的要打包的数据是类似这样的存在。

```bash
[root@soulteary html]# ll
drwxr-xr-x 4 root root     4096 May 14 19:06 domain.com
drwxr-xr-x 4 root root     4096 May 19 01:53 domain.net
drwxr-xr-x 4 root root     4096 May 19 03:35 www.domain.com
```

那么很容易的想到要用`find . -maxdepth 1 -type d -name "*.*"`来获取要打包的文件夹的名称

快速的得到了当前目录下的名字中包含"*.*"的文件夹后，接着要逐条执行打包的话，又会很当然的想到要用sed处理正则，最后把sed处理结果执行一下。

测试一下语句，打印出来先

```bash
[root@soulteary html]# find . -maxdepth 1 -type d -name "*.*" | sed -n 's/\.\/\(.*\)/tar zcvf \1.tar.gz &/p'
tar zcvf domain.com.tar.gz ./domain.com
tar zcvf domain.net.tar.gz ./domain.net
tar zcvf www.domain.com.tar.gz ./www.domain.com
```

看来语句没有问题，那么在最后加上 "sh"，完成调用。

```bash
[root@soulteary html]# find . -maxdepth 1 -type d -name "*.*" | sed -n 's/\.\/\(.*\)/tar zcvf \1.tar.gz &/p |sh'
```

但是现在看到目录下零零散散的一堆文件，不太好吧。

```bash
[root@soulteary html]# ll
drwxr-xr-x 4 root root     4096 May 14 19:06 domain.com
-rw-r--r-- 1 root root  4793496 May 24 14:07 domain.com.tar.gz
drwxr-xr-x 4 root root     4096 May 19 01:53 domain.net
-rw-r--r-- 1 root root 12278664 May 24 14:07 domain.net.tar.gz
```

那么，应该把这些打包的文件当作临时文件处理，最后汇总成一个包。

处理过程和上面的类似，就不详细写了（已经很详细了，汗）。

```bash
echo 'WEBSITE BACKUP SCRIPT BY SOULTEARY v2013.05.24'
echo '1.Clean up prev backup files:'
rm -rf backup
echo '2.Backup process begin:'
mkdir backup
echo '3.Backup each files:'
find . -maxdepth 1 -type d -name "*.*" | sed -n 's/\.\/\(.*\)/tar zcvf \1.tar.gz &/p' |sh
echo '4.Move each backup:'
find . -maxdepth 1 -name "*.tar.gz" | sed -n 's/\(.*\)/mv \1 .\/backup\//p'|sh
echo '5.Combine backup:'
tar zcvf backup.tar.gz backup
echo '6.Done:'
mv backup.tar.gz backup
```

如果你不想要小文件的备份的话，可以在步骤5后直接删除backup文件，那么就只剩下一个大的压缩包了。

