# [WIN]彻底清除已删除文件信息

[![fair](https://attachment.soulteary.com/2011/01/25/fair.jpg "fair")](https://attachment.soulteary.com/2011/01/25/fair.jpg) 

各种安全机构的安全删除工具,无非是多次擦写,但是他们唯独漏掉一个地方. 分区表,虽然文件不能被回复,但是文件名依旧被牢牢的保存在硬盘中, MS其实早对此作出了应对方案:cipher 哼着周董的“暗号”,打开CMD, OR 直接在运行中输入

```bash
cipher /W:C:\
```

C:\是你要fair well数据的完整路径，一般来说我们会全盘清理一下。 所以,你需要修改这个为

```bash
cipher /W:D:\...到E,F,G,等
```

当然你可以写批处理,做循环变量.这个没有试验过,因为cipher的处理速度很慢很慢. 我清理了大概1个T的数据.用了近10个小时. 

fair well,records.

