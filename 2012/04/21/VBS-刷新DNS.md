# [VBS]刷新DNS

桌面发现一个文件夹...貌似是很久以前写的...贴上来备份下

<!-- more -->

```vb
On Error Resume Next

Set objWMIService = GetObject("winmgmts:\\.\root\CIMV2")
Set colItems = objWMIService.ExecQuery("SELECT * FROM Win32_NetworkAdapterConfiguration", "WQL", &h10 + &h20)

For Each objItem In colItems
  Call objItem.SetDNSServerSearchOrder
  Call objItem.EnableDHCP
  Call objItem.RenewDHCPLease
  Call objItem.SetTcpipNetbios(2)
  Call objItem.SetTcpipNetbios(0)
Next
```

