# [问题解决]访问IIS元数据库失败的解决办法

访问IIS元数据库失败的解决办法

```
如果是先安装VS，然后再安装的IIS，用IIS调试写完的.NET程序时出现系统提示：“访问 IIS 元数据库失败”。可能是由于安装次序次序出了问题，解决方法如下：

1.以管理员身份运行CMD，用CD命令进入“C:WINDOWSMicrosoft.NETFrameworkv2.0.50727”后，
输入“aspnet_regiis.exe -i”，稍等片刻，注册成功就解决问题了。

2.如果出现“未能创建 Mutex”的问题，那么：

1.先关闭你的VS

2.打开“C:WINDOWSMicrosoft.NETFrameworkv2.0.50727Temporary ASP.NET Files”
找到你刚才调试的程序的名字的目录删除它。

3.重启IIS服务即可。
```


