# [VB]多叉树统计字符个数

多叉树统计字符个数。 记得很久以前从网上收集的，感谢作者的思路。有时候问题如果复杂，不如不简化。交给计算机来做。 使用方法,将代码保存为Form.frm运行即可。

```vb
VERSION 5.00
Begin VB.Form Form1
Caption = "Form1"
ClientHeight = 5280
ClientLeft = 60
ClientTop = 450
ClientWidth = 7665
LinkTopic = "Form1"
ScaleHeight = 5280
ScaleWidth = 7665
StartUpPosition = 3 '窗口缺省
Begin VB.CommandButton Command2
Caption = "分析"
Height = 465
Left = 45
TabIndex = 0
Top = 4770
Width = 7575
End
Begin VB.ListBox List1
Height = 4200
Left = 45
TabIndex = 3
Top = 45
Width = 7575
End
Begin VB.TextBox txtFileSize
Height = 270
Left = 3960
TabIndex = 2
Top = 4410
Width = 600
End
Begin VB.CommandButton Command3
Caption = "制作随机文本 MB"
Height = 465
Left = 45
TabIndex = 1
Top = 4320
Width = 7575
End
End
Attribute VB_Name = "Form1"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = False
Option Explicit
Dim txtString() As Integer

Private Sub Command2_Click()
'分析字符数量
Dim tim As Single
tim = Timer

Dim words() As Long
Dim i As Long
Dim c As Long
c = UBound(txtString)
Do
words(txtString(i) - 65) = words(txtString(i) - 65) + 1
i = i + 1
Loop Until i > c

MsgBox "分析完成,耗时: " & Timer - tim & " ms"

List1.Clear
For i = 0 To 25
List1.AddItem "字母 " & Chr(i + 65) & " 的数量有 " & words(i) & " 个"
Next

End Sub

Private Sub Command3_Click()

Dim intFileNums As Integer

'生成随机文本
Dim lngIndex As Long
Dim o As Single
o = Timer

Dim strTmp() As Integer

ReDim strTmp(CLng(txtFileSize.Text) * 512& * 1024&)

Dim lngTotal As Long

Randomize

intFileNums = FreeFile
Open "1.txt" For Binary As #intFileNums
Get #intFileNums, , strTmp
Close #intFileNums

MsgBox txtFileSize & " MB 随机文本制作时间: " & Timer - o & " ms"

txtString = strTmp
End Sub
```

