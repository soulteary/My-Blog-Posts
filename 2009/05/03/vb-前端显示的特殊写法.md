# [vb]前端显示的特殊写法

[vb]前端显示的特殊写法

```vb
Option Explicit

Private Declare Function SetWindowPos Lib "user32" (ByVal Hwnd As Long, ByVal hWndInsertAfter As Long, _
ByVal X As Long, ByVal Y As Long, ByVal cx As Long, ByVal cy As Long, ByVal wFlags As Long) As Long

Private Const HWND_TOPMOST = -1
Private Const HWND_NOTOPMOST = -2
Private Const SWP_NOSIZE = &H1
Private Const SWP_NOMOVE = &H2

Public Property Let OnTop(frm As Form, Setting As Boolean)

SetWindowPos frm.Hwnd, IIf(Setting, HWND_TOPMOST, HWND_NOTOPMOST), 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE

End Property
```

任意窗体事件中 ：

```vb
Private Sub Form_Load()
OnTop = True
End Sub
```

