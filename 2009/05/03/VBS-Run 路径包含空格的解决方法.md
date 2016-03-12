# [VBS]Run 路径包含空格的解决方法

[VBS]Run 路径包含空格的解决方法 VBS不能识别长路径，例如 C:\Program Files\abc.exe 需要写成 C:\Progra~1\abc.exe 如果你觉得上面的规则麻烦的话，那么使用一个小窍门可以曲线救国，继续使用完整的长路径。 只要在路径前后添加 chr(34) 即可

```vb
Dim Wsh
Set Wsh = WScript.CreateObject("WScript.Shell")
Wsh.Run chr(34) & "C:\Program Files\abc.exe" & chr(34),,True
Set Wsh=NoThing
WScript.quit
```


