# [VB]发送邮件

一个发送邮件的函数。

```vb
Option Explicit

Private Declare Function ShellExecute _
Lib "shell32.dll" _
Alias "ShellExecuteA" (ByVal hwnd As Long, _
ByVal lpOperation As String, _
ByVal lpFile As String, _
ByVal lpParameters As String, _
ByVal lpDirectory As String, _
ByVal nShowCmd As Long) As Long

Private Const SW_SHOW = 5

Public Function SendEmail(ByVal strMail As String, _
Optional strSubject As String = "Subject", _
Optional strBody As String = "The Content Here.") As Boolean

Dim lngRtn As Long, strTmp As String: lngRtn = 0

strTmp = "mailto:" & strMail & "?subject=" & strSubject & "&body=" & strBody

lngRtn = ShellExecute(Me.hwnd, "open", strTmp, vbNullString, vbNullString, SW_SHOW)

SendEmail = IIf(lngRtn = 0, False, True)

End Function

```

