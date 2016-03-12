# [VB]简单的下雨屏保

简单的下雨屏保 说明,整理硬盘看到，修改为动态创建控件，使用本代码，你只需要修改窗口名称为frmMain，以及设置窗口样式为无边框，接下来直接F5或者运行即可。

```vb
'by promiseforever.com
'先设置窗口名称为frmMain
'再设置窗口样式为无边框

Option Explicit

Dim X(1 To 100) As Long, Y(1 To 100) As Long, pace(1 To 100) As Integer, size(1 To 100) As Integer

Dim WithEvents tmrFir As Timer

Private Sub Form_Activate()

    Randomize

    Dim i As Byte, w As Long, h As Long, p As Integer, s As Integer

    For i = 1 To 100

        w = Int(frmMain.Width * Rnd)
        h = Int(frmMain.Height * Rnd)
        p = Int(500 - (Int(Rnd * 499)))
        s = 25 * Rnd

        X(i) = w
        Y(i) = h
        pace(i) = p
        size(i) = s
    Next

End Sub

Private Sub Form_Click()
    Call Form_Quit
End Sub

Private Sub Form_Load()

    With Me
        .AutoRedraw = True
        '.BorderStyle = 0 '在属性窗口设置
        .BackColor = &H0
        .ForeColor = &HE0E0E0
        .Left = 0
        .Top = 0
        .Width = Screen.Width
        .Height = Screen.Height

    End With

    Set tmrFir = Me.Controls.Add("VB.Timer", "tmrFir")

    With tmrFir
        .Interval = 10
        .Enabled = True
    End With

End Sub

Private Sub Form_Quit()
    Unload Me: Set frmMain = Nothing: End
End Sub

Private Sub tmrFir_Timer()

    Dim i As Byte

    For i = 1 To 100

        Circle (X(i), Y(i)), size(i), BackColor

        Y(i) = Y(i) + pace(i)

        If Y(i) >= frmMain.Height Then Y(i) = 0: X(i) = Int(frmMain.Width * Rnd)
        Circle (X(i), Y(i)), size(i)
    Next

End Sub
```

