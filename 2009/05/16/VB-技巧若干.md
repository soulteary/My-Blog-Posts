# [VB]技巧若干

VB技巧若干,当复习了,贴上来备忘...高手自觉莫入。

```vb
'格式化软盘的函数
Private Function FormatDriver() As Boolean

Dim strCmd As String

Dim lngRtn As Long

strCmd = "rundll32.exe shell32.dll,SHFormatDriver"

lngRtn = Shell(strCmd, vbNormalFocus)

If lngRtn = 0 Then

FormatDriver = False

Else

FormatDriver = True

End If

End Function
```

```vb
'显示隐藏任务栏
Option Explicit

Private Declare Function FindWindow _
Lib "user32" _
Alias "FindWindowA" (ByVal lpClassName As String, _
ByVal lpWindowName As String) As Long

Private Declare Function SetWindowPos _
Lib "user32" (ByVal hwnd As Long, _
ByVal hWndInsertAfter As Long, _
ByVal x As Long, _
ByVal y As Long, _
ByVal cx As Long, _
ByVal cy As Long, _
ByVal wFlags As Long) As Long

Private Const SWP_HIDEWINDOW As Long = &H80

Private Const SWP_SHOWWINDOW As Long = &H40

Private Const FIR_TASKBAR As String = "Shell_TrayWnd"

Dim lngRtn As Long, lngTmp As Long

Private Function HideTaskBar() As Boolean

lngTmp = FindWindow(FIR_TASKBAR, "")
lngRtn = SetWindowPos(lngTmp, 0, 0, 0, 0, 0, SWP_HIDEWINDOW)

If lngRtn = 0 Then
HideTaskBar = False
Else
HideTaskBar = True
End If

End Function

Private Function ShowTaskBar() As Boolean

lngTmp = FindWindow(FIR_TASKBAR, "")
lngRtn = SetWindowPos(lngTmp, 0, 0, 0, 0, 0, SWP_SHOWWINDOW)

If lngRtn = 0 Then
ShowTaskBar = False
Else
ShowTaskBar = True
End If

End Function
```

```vb
'窗体总在前端
Option Explicit

Private Const SWP_NOSIZE As Long = &H1

Private Const SWP_NOMOVE As Long = &H2

Private Const HWND_TOP As Long = 0

Private Const HWND_NOTOPMOST As Long = -2

Private Const HWND_BOTTOM As Long = 1

Private Const HWND_TOPMOST As Long = -1

Private Declare Function SetWindowPos _
Lib "user32" (ByVal hwnd As Long, _
ByVal hWndInsertAfter As Long, _
ByVal x As Long, _
ByVal y As Long, _
ByVal cx As Long, _
ByVal cy As Long, _
ByVal wFlags As Long) As Long

Private Sub PutOnTop(frm As Form)

Call SetWindowPos(frm.hwnd, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE)

End Sub

Private Sub PutNormal(frm As Form)

Call SetWindowPos(Me.hwnd, HWND_NOTOPMOST, 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE)

End Sub
```

```vb
'限制鼠标移动范围以及正确的解除鼠标限制
Option Explicit

Private Declare Function ClipCursor Lib "user32" (lpRect As Any) As Long

Private Declare Function GetWindowRect Lib "user32" (ByVal hwnd As Long, lpRect As RECT) As Long

Private Type RECT
Left As Long
Top As Long
Right As Long
Bottom As Long
End Type

Dim My_Rect As RECT

Dim lngTmp As Long, lngRtn As Long

'限制鼠标在窗体内
Call GetWindowRect(Me.hwnd, My_Rect)
Call ClipCursor(My_Rect)

'解除限制
lngRtn = GetDesktopWindow()
Call GetWindowRect(lngRtn, My_Rect)
Call ClipCursor(My_Rect)
```

窗体透明

```vb
Option Explicit

Private Declare Function SetWindowLong _
Lib "user32" _
Alias "SetWindowLongA" (ByVal hwnd As Long, _
ByVal nIndex As Long, _
ByVal dwNewLong As Long) As Long

Private Const WS_EX_TRANSPARENT As Long = &H20

Private Const GWL_EXSTYLE As Long = (-20)

Public Sub makeTransparent(frm As Form)
Call SetWindowLong(frm.hwnd, GWL_EXSTYLE, WS_EX_TRANSPARENT)
End Sub
```

创建椭圆或圆形窗口

```vb
Option Explicit

Public Declare Function CreateEllipticRgn _
Lib "gdi32" (ByVal X1 As Long, _
ByVal Y1 As Long, _
ByVal X2 As Long, _
ByVal Y2 As Long) As Long

Public Declare Function SetWindowLong _
Lib "user32" _
Alias "SetWindowLongA" (ByVal hwnd As Long, _
ByVal nIndex As Long, _
ByVal dwNewLong As Long) As Long

Private Form_Load()

'注：hRgn=CreateEllipticRgn(0,0,300,200)中的四个参数分别是椭圆窗体的外切矩形的左上角（0，0）和右下角（300，200）的坐标，
'根据这我们只要将它的两条外切边设为相等则可以绘出圆形的窗体了!
Dim lngRtn As Long

lngRtn = CreateEllipticRgn(0, 0, 300, 200)
Call SetWindowRgn(Me.hwnd, lngRtn, True)
End Sub
```

闪烁窗体

```vb
Option Explicit

Private Declare Function FlashWindow _
Lib "user32" (ByVal hwnd As Long, _
ByVal bInvert As Long) As Long

Private Sub Timer1_Timer()
Call FlashWindow(Me.hwnd, 1)
End Sub
```

