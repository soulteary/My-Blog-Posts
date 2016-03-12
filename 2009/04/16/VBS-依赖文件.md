# [VBS]依赖文件

VBS依赖的文件

`scrrun.dll, WSHom.Ocx,shell32.dll`

如果一些涉及 Wscript.shell ，Scripting.FileSystemObject等方面的脚本无法正常运行，很有可能是系统关闭了此三个文件，可以通过如下命令逐一进行注册。

```dos
regsvr32 regsvr32 scrrun.dll /s
regsvr32 WSHom.Ocx /s
regsvr32 shell32.dll /s
```

