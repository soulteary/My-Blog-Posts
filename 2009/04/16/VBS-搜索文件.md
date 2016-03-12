# [VBS]搜索文件

VBS搜索文件

```vb
Dim keyWord, DirTotal, TimeSpend, FileTotal, Fso, outFile, txtResult, txtPath, sPath    

Set Fso = wscript.CreateObject("scripting.filesystemobject")    

FileTotal = 0    
DirTotal = 0    
sPath = left(Wscript.ScriptFullName,len(Wscript.ScriptFullName)-len(Wscript.ScriptName))    
txtPath = trim(inputbox("请输入起始目录:","文件搜索",sPath))    
keyWord = LCase(inputbox("请输入搜索关键字:","文件搜索","mp3"))    

set outFile = Fso.createtextfile(sPath & "SearchResult.txt")    

outFile.writeline "开始搜索..."    
outFile.writeline "起启目录:" & txtPath    
TimeSpend = Timer    

myFind txtPath    

TimeSpend = round(Timer - TimeSpend,2)    

txtResult = "搜索完成!" & vbCrLf & "共找到文件:" & FileTotal & "个." & vbCrLf & "共搜索目录:" & DirTotal & "个." & vbCrLf & "用时:" & TimeSpend & "秒."    
outFile.write txtResult    
msgbox txtResult    

outFile.close    
set outFile = nothing    
set Fso = nothing    

Sub myFind(ByVal thePath)    

Dim fso, myFolder, myFile, curFolder    
Set fso = wscript.CreateObject("scripting.filesystemobject")    
Set curFolders = fso.getfolder(thePath)    
DirTotal = DirTotal + 1    
If curFolders.Files.Count > 0 Then    
For Each myFile In curFolders.Files    
If InStr(1, LCase(myFile.Name), keyWord) > 0 Then    
outFile.WriteLine FormatPath(thePath) & "" & myFile.Name   
FileTotal = FileTotal + 1   
End If   
Next   
End If   

If curFolders.subfolders.Count > 0 Then   
For Each myFolder In curFolders.subfolders   
myFind FormatPath(thePath) & "" & myFolder.Name     
Next   
End If   

End Sub   

Function FormatPath(ByVal thePath)   

thePath = Trim(thePath)   
FormatPath = thePath   
If Right(thePath, 1) = "" Then FormatPath = Mid(thePath, 1, Len(thePath) - 1)    

End Function   
``` 

