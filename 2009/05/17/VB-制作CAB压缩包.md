# [VB]制作CAB压缩包

如何使用VB制作CAB压缩包。 引用 COM MakeCab 1.0 Type Library以及FSO对象模型。代码如下：

```vb
Option Explicit
Private Function IsFileExist(strPath As String) As Boolean
Dim objFile As New FileSystemObject
IsFileExist = objFile.FileExists(strPath)
End Function
Private Function GetFileName(strPath As String) As String
Dim objFile As New FileSystemObject
GetFileName = objFile.GetFileName(strPath)
End Function
Private Sub cmdCommand1_Click()
Dim strPath As String, strFile As String
strPath = App.Path & "\1.cab"
strFile = VBA.Environ("WinDir")
If Right$(strFile, 1) <> "\" Then strFile = strFile & "\system32\notepad.exe"
If IsFileExist(strFile) = True Then
Call MakeCab(strPath, strFile)
MsgBox "DONE."
Else
MsgBox "The File Not Exist."
End If
End Sub
Public Sub MakeCab(ByVal strCabName As String, ByVal strFile As String)
'引用 : COM MakeCab 1.0 Type Library
If IsFileExist(strCabName) = True Then
MsgBox "The CAB File Is Exist."
Exit Sub
End If
Dim cabMaker As New COMMKCABLib.MakeCab
'Windows 2000 : cabMaker.CreateCab(cabFile, False, 0)
Call cabMaker.CreateCab(strCabName, False, 0, False)
Dim FileName As String
FileName = GetFileName(strFile)
Call cabMaker.AddFile(strFile, FileName)
Call cabMaker.CloseCab
End Sub
```

