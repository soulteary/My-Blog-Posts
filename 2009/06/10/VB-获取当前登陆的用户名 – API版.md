# [VB]获取当前登陆的用户名 – API版

先做一个伏笔,接下来会发布一些看似不沾边的东西...

但是组合起来又是什么呢。

```vb
Option Explicit

Private Declare Function GetUserName _
                Lib "advapi32.dll" _
                Alias "GetUserNameA" (ByVal lpBuffer As String, _
                                      nSize As Long) As Long

Private Function Fir_GetUserName() As String

Const Error As String = "Get User Name Error."
Dim lngLen As Long, lngRet As Long, strRet As String

lngLen = &H400: lngRet = 0: strRet = Space$(lngLen)

lngRet = GetUserName(strRet, lngLen)

If lngRet = 0 Then
    Fir_GetUserName = Error
Else
    Fir_GetUserName = Left$(strRet, lngLen)
End If

End Function

Private Sub Form_Load()

Dim txtUser As TextBox

Set txtUser = Me.Controls.Add("VB.TextBox", "txtUser")

With txtUser
    .Top = .Left = 0
    .Width = Me.Width
    .Height = Me.Height
    .Visible = True
End With

With Me
.Show
End With

    txtUser.Text = Fir_GetUserName()

End Sub
```

