# 为XP的USB添加“只读属性”

Windows XP中有一个非常不错的新功能：为USB存储设备添加“只读”属性。 

## 正文

具体实现方法如下：

进入注册表编辑器，找到

```reg
HKEY_LOCAL_ MACHINESYSTEMCurrentControlSetControlStorageDevicePolicies
```

如果没有该项请新建一个，然后在右侧窗口中新建一个名为WriteProtect的DWORD值，将该键的值赋为“1”，退出程序或进行刷新即可生效。

以后，如果向USB存储设备写入数据，会弹出图2所示的对话框，提示说“介质受写入保护”，这样就不用担心所保存的重要数据遭误操作了。

如果需要解除只读属性，只需再次进入注册表编辑器，将WriteProtect值重新赋为“0”就行了。


