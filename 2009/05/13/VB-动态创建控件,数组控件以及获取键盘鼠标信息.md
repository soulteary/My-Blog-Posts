# [VB]动态创建控件，数组控件以及获取键盘鼠标信息

```vb
Option Explicit
Private Declare Function GetCaretBlinkTime Lib "user32" () As Long
Private Const SPI_GETKEYBOARDSPEED = 10
Private Const SPI_GETKEYBOARDDELAY = 22
Private Declare Function SystemParametersInfo _
Lib "user32" _
Alias "SystemParametersInfoA" (ByVal uAction As Long, _
ByVal uParam As Long, _
lpvParam As Any, _
ByVal fuWinIni As Long) As Long
Private Declare Function GetKeyboardType Lib "user32" (ByVal nTypeFlag As Long) As Long
Private Declare Function GetDoubleClickTime Lib "user32" () As Long
Private Const SM_CMOUSEBUTTONS = 43
Private Declare Function GetSystemMetrics Lib "user32" (ByVal nIndex As Long) As Long
Private Const SM_MOUSEPRESENT = 19
Dim WithEvents cmdGetInfo As VB.CommandButton
Dim WithEvents tmrFlash As VB.Timer
Dim shpShape As VB.Shape
Dim lblInfo() As VB.Label
Private Sub Form_Load()
Me.Show
Set cmdGetInfo = Controls.Add("VB.CommandButton", "cmdGetInfo", Me)
With cmdGetInfo
.Height = 1000
.Width = 1000
.Left = (Me.Width - .Width) / 2
.Top = (Me.Height - .Height) / 2
.Caption = "Click Me To Get Info."
.Visible = True
End With
Set tmrFlash = Controls.Add("VB.Timer", "tmrFlash", Me)
Set shpShape = Controls.Add("VB.Shape", "shpShape", Me)
With shpShape
.BackColor = vbBlack
.FillColor = vbBlack
.BackStyle = 1
.Left = 300
.Top = 300
.Width = 150
.Height = 300
.Visible = False
End With
Dim bIndex As Byte: bIndex = 4
ReDim Preserve lblInfo(1 To bIndex)
For bIndex = 1 To 4
Set lblInfo(bIndex) = Controls.Add("VB.Label", "lblInfo" & CStr(bIndex), Me)
With lblInfo(bIndex)
.Caption = ""
.Left = 200
.Top = 300 * bIndex
.AutoSize = True
.Visible = True
End With
Next
End Sub

Private Sub cmdGetInfo_Click()
Dim lngReturn As Long, strReturn As String
lngReturn = GetKeyboardType(0)
Select Case lngReturn
Case Is = 1
strReturn = "PC or compatible 83－key keyboard"
Case Is = 2
strReturn = "Olivetti 102－key keyboard"
Case Is = 3
strReturn = "AT or compatible 84－key keyboard"
Case Is = 4
strReturn = "Enhanced(IBM) 101－102－key keyboard"
Case Is = 5
strReturn = "Nokia 1050 keyboard"
Case Is = 6
strReturn = "Nokia 9140 keyboard"
Case Is = 7
strReturn = "Japanese keyboard"
Case Else
strReturn = "Unknown."
End Select
MsgBox strReturn, vbInformation + vbOKOnly, "Keyboard Type:"
Dim lngTmp As Long
Call SystemParametersInfo(SPI_GETKEYBOARDDELAY, 0, lngReturn, 0)
strReturn = "Keyboard Repeat Delay = " & lngReturn & " Seconds"
Call SystemParametersInfo(SPI_GETKEYBOARDSPEED, 0, lngReturn, 0)
strReturn = strReturn & vbNewLine & "Keyboard Repeat Speed = " & lngReturn & " characters per second."
lngReturn = GetCaretBlinkTime
With tmrFlash
.Interval = lngReturn
.Enabled = True
End With
strReturn = strReturn & vbNewLine & "Caret Flash Speed = " & lngReturn & "ms"
lngReturn = GetSystemMetrics(SM_MOUSEPRESENT)
If lngReturn = 1 Then
lngReturn = GetSystemMetrics(SM_CMOUSEBUTTONS)
strReturn = strReturn & vbNewLine & "Standard Mouse Present with " & lngReturn & " buttons."
Else
strReturn = strReturn & vbNewLine & "No Mouse Present."
Exit Sub
End If
lngReturn = GetDoubleClickTime
strReturn = strReturn & vbNewLine & "Double Click Speed = " & lngReturn & "ms"
MsgBox strReturn
End Sub
Private Sub tmrFlash_Timer()
shpShape.Visible = Not shpShape.Visible
End Sub

```

