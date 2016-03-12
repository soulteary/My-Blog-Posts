# [vb]TextOut例子

```vb
Option Explicit   

Private Declare Function GetDC Lib "user32" (ByVal hwnd As Long) As Long  
Private Declare Function TextOut Lib "gdi32" Alias "TextOutA" (ByVal hdc As Long, ByVal x As Long, ByVal y As Long, ByVal lpString As String, ByVal nCount As Long) As Long  
Private Declare Function FindWindow Lib "user32" Alias "FindWindowA" (ByVal lpClassName As String, ByVal lpWindowName As String) As Long  
Private Declare Function ReleaseDC Lib "user32" (ByVal hwnd As Long, ByVal hdc As Long) As Long  

Private Sub cmdDrawA_Click()   
Dim lngDC As Long  
Dim strTmp As String  
lngDC = GetDC(&H0)   
strTmp = "凉风有信  秋月无边"  
Call TextOut(lngDC, 100, 100, strTmp, LenB(strTmp) - 1)   
Call ReleaseDC(Me.hwnd, lngDC)   
End Sub  

Private Sub cmdDrawB_Click()   
Dim lngHwnd As Long  
Dim lngDC As Long  
strTmp = "凉风有信  秋月无边"  
lngHwnd = FindWindow("Notepad", "无标题 - 记事本")   
If lngHwnd = 0 Then MsgBox "0"  
lngDC = GetDC(lngHwnd)   
Call TextOut(lngDC, 10, 10, strTmp, LenB(strTmp) - 1)   
End Sub  

Private Sub cmdDrawC_Click()   
Dim lngHwnd As Long  
Dim lngDC As Long  
strTmp = "凉风有信  秋月无边"  
lngDC = GetDC(Me.hwnd)   
Call TextOut(lngDC, 10, 10, strTmp, LenB(strTmp) - 1)   
End Sub
```

