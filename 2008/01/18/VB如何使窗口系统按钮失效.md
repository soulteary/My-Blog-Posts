# VB如何使窗口系统按钮失效

话不多说，直接上代码。

```vb
Private Declare Function GetSystemMenu _
                Lib "user32" (ByVal hwnd As Long, _
                              ByVal bRevert As Long) As Long

Private Declare Function RemoveMenu _
                Lib "user32" (ByVal Hmenu As Long, _
                              ByVal nPosition As Long, _
                              ByVal wFlags As Long) As Long

Private Const MF_REMOVE = &H1000&

Private Const SC_CLOSE = &HF060&

Private Const SC_MINIMIZE = &HF020&

Private Const SC_MAXIMIZE = &HF030&

Private Sub Command1_Click()
    Call CloseMenu
End Sub

Private Sub CloseMenu()

    Dim Hmenu As Long

    Hmenu = GetSystemMenu(Me.hwnd, 0)
    Call RemoveMenu(Hmenu, SC_CLOSE, MF_REMOVE)
    Call RemoveMenu(Hmenu, SC_MINIMIZE, MF_REMOVE)
    Call RemoveMenu(Hmenu, SC_MAXIMIZE, MF_REMOVE)
End Sub
```


