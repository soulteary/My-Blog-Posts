# [vb]如何检测文件是否存在[6种方法]

如果你还不清楚VB中有那些方法可以检测一个文件是否存在，那么看看吧。

通常我们检测文件存在都是使用Dir，除了Dir之外还可以使用SHFileExists、GetFileAttributes、PathFileExists、Open打开文件、以及利用FSO检测文件是否存在 [要记得引用MS Scripting RunTime]。


```vb

Option Explicit
 
    Private Declare Function SHFileExists Lib "shell32" Alias "#45" (ByVal szPath As String) As Long
    Private Declare Function GetFileAttributes Lib "kernel32" Alias "GetFileAttributesA" (ByVal lpFilePath As String) As Long
    Private Declare Function PathFileExists Lib "shlwapi.dll" Alias "PathFileExistsA" (ByVal pszPath As String) As Long
    Private Const vbAllFileAttrib = vbNormal + vbReadOnly + vbHidden + vbSystem + vbVolume + vbDirectory
 
 
Private Sub Main()
 
MsgBox CheckFileExists("C:AUTOEXEC.BAT", 1) & vbNewLine & _
       CheckFileExists("C:AUTOEXEC.scr", 2) & vbNewLine & _
       CheckFileExists("C:AUTOEXEC.pif", 3) & vbNewLine & _
       CheckFileExists("C:AUTOEXEC.com", 4) & vbNewLine & _
       CheckFileExists("C:AUTOEXEC.exe", 0)
 
End Sub
 
Public Function CheckFileExists(FilePath As String, Index As Byte) As Boolean
 
    On Error GoTo Err
      
    If Len(FilePath) < 2 Then CheckFileExists = False: Exit Function
      
    Select Case Index
 
        Case 0
          
            If Dir$(FilePath, vbAllFileAttrib) <> vbNullString Then CheckFileExists = True
 
        Case 1
          
        Dim FileNum As Integer: FileNum = FreeFile
        Open FilePath For Input As #FileNum: Close #FileNum: CheckFileExists = True
 
        Case 2
          
            If Str$(GetFileAttributes(FilePath)) <> -1 Then CheckFileExists = True
 
        Case 3
          
            CheckFileExists = CBool(PathFileExists(FilePath))
 
        Case 4
 
            If SHFileExists(FilePath) <> 0 Then CheckFileExists = True
      
    End Select
 
    Exit Function
 
Err:
 
    CheckFileExists = False
      
End Function
```

利用FSO检测文件是否存在 [要记得引用MS Scripting RunTime]

```vb
Function FileExist(FilePath As String)
 
    Dim Fso As New FileSystemObject
 
    If Fso.FileExists(FilePath) = True Then
        FileExist = True
    Else
        FileExist = False
    End If
 
    Set Fso = Nothing
End Function
 
Function FolderExist(FolderPath As String)
 
    Dim Fso As New FileSystemObject
 
    If Fso.FolderExists(FolderPath) = True Then
        FolderExist = True
    Else
        FolderExist = False
    End If
 
    Set Fso = Nothing
End Function
```

