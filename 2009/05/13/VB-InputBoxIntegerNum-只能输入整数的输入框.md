# [VB]InputBoxIntegerNum-只能输入整数的输入框

一个利用InputBox求和的例子。重新写了个小函数。便于限制输入为整数数字。 **特别感谢来自台湾的朋友GIDIN的指正~！** ,多增加了几个判断,但是应该不至于影响速度。

```vb
Option Explicit

Private Sub Command1_Click()

End Sub

Private Function InputIntegerNumBox(Optional strLabel As String = "请输入一个数字", Optional strCaption As String = "程序提示") As Long

On Error Resume Next

Dim strTmp As String, bPoint As Byte

Do

If strTmp = "Err" Then strLabel = "Input A Integer,Plz."
strTmp = InputBox(strLabel, strCaption)

If InStr(strTmp, "-") Then
If Left$(strTmp, 1) <> "-" Then strTmp = "Err"
End If

bPoint = InStr(strTmp, ".")
If bPoint Then

bPoint = bPoint + 1

If InStr(bPoint, strTmp, ".") Then

strTmp = "Err"

Else

If Val(Mid(strTmp, bPoint)) <> 0 Then strTmp = "Err"

End If

End If

Loop Until IsNumeric(strTmp) = True

InputIntegerNumBox = Val(strTmp)

End Function

Private Sub cmdCommand1_Click()

Dim lngNumA As Long, lngNumB As Long

Dim lngSum As Long

lngNumA = InputIntegerNumBox("Input The 1st Num ,Plz.", "Fir Says:")
lngNumB = InputIntegerNumBox("Input The 2nd Num ,Plz.", "Fir Says:")

lngSum = lngNumA + lngNumB

MsgBox "The Result is " & lngSum

End Sub

```

