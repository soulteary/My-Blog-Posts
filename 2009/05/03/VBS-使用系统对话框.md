# [VBS]使用系统对话框

在VBS脚本设计中，如果能使用windows提供的系统对话框，可以简化脚本的使用难度，使脚本人性化许多，很少有人使用，但VBS并非不能实现这样的功能，方法当然还是利用COM对象。

<!-- more -->

1、SAFRCFileDlg.FileSave对象：
属性有：FileName — 指定默认文件名
FileType —指定文件扩展名
OpenFileSaveDlg — 显示文件保存框体方法

2、SAFRCFileDlg.FileOpen 对象：
FileName — 默认文件名属性；
OpenFileOpenDlg — 显示打开文件框体方法。

3、UserAccounts.CommonDialog对象：Filter — 扩展名属性（"vbs File|*.vbs|All Files|*.*"）；
FilterIndex — 指定
InitialDir — 指定默认的文件夹
FileName — 指定的文件名
Flags — 对话框的类型
Showopen方法：

以下有三个例子：
例一：保存文件

```vb
Set objDialog = CreateObject("SAFRCFileDlg.FileSave")   
  
Set objFSO = CreateObject("Scripting.FileSystemObject")   
  
objDialog.FileName = "test"  
  
objDialog.FileType = ".txt"  
  
intReturn = objDialog.OpenFileSaveDlg   
  
If intReturn Then  
  
objFSO.CreateTextFile(objDialog.FileName &amp; objdialog.filetype)   
  
Else  
  
Wscript.Quit   
  
End If  
```

<!--more-->

注意：1、SAFRCFileDlg.FileSave对象仅仅是提供了一个方便用户选择的界面，本身并没有保存文件的功能，保存文件还需要使用FSO对象来完成。2、用FileType属性来指定默认的文件类型。3、在调用OpenFileSaveDlg方法时，最好把返回值保存到一变量中，用它可以判断用户按下的是确定还是取消。

例二：.打开文件

```vb
set objFile = CreateObject("SAFRCFileDlg.FileOpen")   
  
intRet = objFile.OpenFileOpenDlg   
  
if intret then   
  
msgbox "文件打开成功！文件名为：" &amp; objFile.filename   
  
else   
  
wscript.quit   
  
end if   
```

例三：比较复杂的打开文件对话框

```vb
Set objDialog = CreateObject("UserAccounts.CommonDialog")   
  
objDialog.Filter = "vbs File|*.vbs"  
  
objDialog.InitialDir = "c:"  
  
tfile=objDialog.ShowOpen   
  
if tfile then    
  
strLoadFile = objDialog.FileName   
  
msgbox strLoadFile   
  
else   
  
wscript.quit   
  
end if
```

说明：在脚本中加入 objDialog.Flags = &amp;H020 看看会出现什么结果。

