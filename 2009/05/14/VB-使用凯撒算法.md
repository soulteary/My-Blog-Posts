# [VB]使用凯撒算法

花了点时间写了一个凯撒算法的实现[人懒,不做中文的移位算法咯,其实可以演变出来十几种其他的算法,只要设置私匙以及字码位就可以了] 

动态创建控件，你只需要把代码复制到一个窗体中就可以了。

```vb
Option Explicit

Dim WithEvents txtInput As VB.TextBox

Dim WithEvents txtOutput As VB.TextBox

Dim WithEvents txtNums As VB.TextBox

Dim WithEvents cmdGen As VB.CommandButton

Dim intLetterL(97 To 122) As Integer

Dim intLetterU(65 To 90) As Integer

Dim lngIndex As Long

Private Sub Form_Load()

With Me
.Caption = "凯撒加密算法 示例 by 苏洋"
.Width = 4800
.Height = 3600

End With

Me.Show

Set txtInput = Controls.Add("VB.TextBox", "txtInput", Me)

With txtInput

.Top = 500
.Left = 500
.Height = 500
.Width = Me.Width - 1000

.Visible = True
End With

Set txtOutput = Controls.Add("VB.TextBox", "txtOutput", Me)

With txtOutput

.Top = 1500
.Left = 500
.Height = 500
.Width = Me.Width - 1000

.Visible = True
End With

Set txtNums = Controls.Add("VB.TextBox", "txtNums", Me)

With txtNums

.Top = 2500
.Left = 500
.Height = 300
.Width = 500
.Text = 3
.Visible = True
End With

Set cmdGen = Controls.Add("VB.CommandButton", "cmdGen", Me)

With cmdGen
.Top = 2500
.Left = 1200
.Width = 3300
.Height = 300
.Caption = "在左边输入要偏移的量,然后点击按钮"
.Visible = True
End With

Call cmdGen_Click

End Sub

Private Sub cmdGen_Click()
Call IntArry
Call EncodeLetterU
Call EncodeLetterL
End Sub

Private Sub IntArry()

For lngIndex = 65 To 90

intLetterU(lngIndex) = lngIndex

Next

For lngIndex = 97 To 122

intLetterL(lngIndex) = lngIndex

Next

End Sub

Private Sub EncodeLetterU(Optional MoveL As Boolean = True)

Dim lngTmp As Long

Select Case MoveL

Case True

For lngIndex = 65 To 90
lngTmp = lngIndex + Val(txtNums.Text)

If lngTmp > 90 Then
'- 90 + 65
intLetterU(lngIndex) = lngTmp - 25
Else

intLetterU(lngIndex) = lngTmp

End If

Next

Case False

For lngIndex = 65 To 90
lngTmp = lngIndex - Val(txtNums.Text)

If lngTmp < 65 Then
'90-65-lngtmp
intLetterU(lngIndex) = 25 - lngTmp
Else

intLetterU(lngIndex) = lngTmp

End If

Next

End Select

End Sub

Private Sub EncodeLetterL(Optional MoveL As Boolean = True)

Dim lngTmp As Long

Select Case MoveL

Case True

For lngIndex = 97 To 122
lngTmp = lngIndex + Val(txtNums.Text)

If lngTmp > 122 Then
'- 122 + 97
intLetterL(lngIndex) = lngTmp - 25
Else

intLetterL(lngIndex) = lngTmp

End If

Next

Case False

For lngIndex = 97 To 122
lngTmp = lngIndex - Val(txtNums.Text)

If lngTmp < 97 Then
'122-97-lngtmp
intLetterL(lngIndex) = 25 - lngTmp
Else

intLetterL(lngIndex) = lngTmp

End If

Next

End Select

End Sub

Private Sub txtInput_KeyDown(KeyCode As Integer, Shift As Integer)

txtOutput = txtInput

End Sub

Private Sub txtInput_Change()

txtOutput = ""

Dim lngTmp As Long, lngLen As Long, intCodes As Integer, strTmp As String

lngLen = Len(txtInput)

For lngTmp = 1 To lngLen

strTmp = Mid$(txtInput, lngTmp, 1)
intCodes = Asc(strTmp)

Select Case intCodes

Case 65 To 90
txtOutput = txtOutput & Chr(intLetterU(intCodes))

Case 97 To 122
txtOutput = txtOutput & Chr(intLetterL(intCodes))
End Select

Next

End Sub

Private Sub txtInput_KeyPress(KeyAscii As Integer)

If Not Chr(KeyAscii) Like "[a-zA-Z]" Then KeyAscii = 0

End Sub
```

```text
恺撒加密算法

“恺撒密码”相传是古罗马恺撒大帝用来保护重要军情的加密手段。
它主要是一种使用字符替换的加密算法，通过将字母顺序依次推后数位[原始是3位]，混淆原始密文来起到加密的作用。

原始信息如下：
RETURN TO ROME
密文信息如下“
UHWXUA WR URPH 

这样无法从字面上直接看出信息的内容了。
这种加密方法还可以依据移位的不同产生新的变化，将每个字母左19位，就产生这样一个明密对照表：

明:A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
密:T U V W X Y Z A B C D E F G H I J K L M N O P Q R S

在这个加密表下，明文与密文的对照关系就变成：

明文：THE FAULT, DEAR BRUTUS, LIES NOT IN OUR STARS BUT IN OURSELVES.
密文：MAX YTNEM, WXTK UKNMNL, EBXL GHM BG HNK LMTKL UNM BG HNKLXEOXL.
```

