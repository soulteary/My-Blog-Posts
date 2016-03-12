# [VB]生成不重复数

百度知道上的一个人的问题， VB 随机产生1-36之间的7个互不相同的数，输入7个互不相同的数，提示猜中数的个数 随机产生7个数后，点击按钮才显示。用7个文本框来输入你猜的数，点击按钮，用msgbox显示猜中的个数。（输入的7个数不能重复，或者大于36） 我用动态数组做了,感觉还可以,或许某些地方啰嗦了。[没有添加强制输入字符为数字，就一两句话,本人懒不写了。] 使用方法，把所有的代码复制，粘贴到一个新的窗口中，直接运行，就啥都有了。

```vb
Option Explicit

Dim Codes(1 To 7) As Byte

Dim txtInput() As VB.TextBox

Dim WithEvents cmdCheck As VB.CommandButton

Dim WithEvents cmdReGen As VB.CommandButton

Dim WithEvents cmdShows As VB.CommandButton

Dim Finish As Boolean

Private Sub cmdCheck_Click()

Dim intIndex As Integer

Static ErrTimes As Byte

If Finish = False Then

ErrTimes = ErrTimes + 1

For intIndex = 1 To 7

If Val(txtInput(intIndex)) = Codes(intIndex) Then

txtInput(intIndex) = Codes(intIndex)

MsgBox "恭喜,第" & intIndex & " 个猜对了。"

txtInput(intIndex).Enabled = False

End If

Next

End If

Dim txtTmp

For Each txtTmp In txtInput

If txtTmp.Enabled = True Then

Exit For

Else
MsgBox "完成猜测,猜测次数" & ErrTimes
Finish = True

Exit Sub

End If

Next

MsgBox "猜测次数" & ErrTimes

End Sub

Private Sub cmdReGen_Click()

Dim bIndex As Byte

For bIndex = 1 To 7

txtInput(bIndex).Text = ""

Next

Call GenCodes
Finish = False
MsgBox "重新生成完毕。"

End Sub

Private Sub cmdShows_Click()

Dim bIndex As Byte

For bIndex = 1 To 7

txtInput(bIndex) = Codes(bIndex)
Next

Finish = True
MsgBox "下次继续努力吧。"

End Sub

Private Sub Form_Load()

Finish = False

With Me

.Height = 3000

.Show
End With

Dim bIndex As Byte: bIndex = 7

ReDim Preserve txtInput(1 To bIndex) As VB.TextBox

Set cmdCheck = Controls.Add("VB.CommandButton", "cmdCheck", Me)

With cmdCheck
.Caption = "检验7个数字"
.Top = 2100
.Height = 250
.Left = 1000
.Width = 3300
.Visible = True
End With

Set cmdReGen = Controls.Add("VB.CommandButton", "cmdReGen", Me)

With cmdReGen
.Caption = "重新生成"
.Top = 1800
.Height = 250
.Left = 1000
.Width = 3300
.Visible = True
End With

Set cmdShows = Controls.Add("VB.CommandButton", "cmdShows", Me)

With cmdShows
.Caption = "显示结果"
.Top = 1500
.Height = 250
.Left = 1000
.Width = 3300
.Visible = True
End With

For bIndex = 1 To 7

Set txtInput(bIndex) = Controls.Add("VB.TextBox", "txtInput" & CStr(bIndex), Me)

With txtInput(bIndex)
.Text = ""
.Top = 300 * bIndex
.Height = 230
.Left = 300
.Width = 500
.Visible = True
End With

Next

Call GenCodes
End Sub

Private Sub GenCodes()

Dim bTmp As Byte, bIndex As Byte, bCode As Variant

For bIndex = 1 To 7
Codes(bIndex) = 0
Next

For bIndex = 1 To 7

bTmp = Rnd * 36 + 1

For Each bCode In Codes

Do

If bTmp = bCode Then bTmp = 0

Loop Until bTmp > 0

Next

Codes(bIndex) = bTmp

Next

End Sub
```

