# [VB]鸡兔同笼解法

```text
写了一个鸡兔同笼的算法，一个是递归枚举，一个是公式大法。
使用自写函数以及Do循环过滤不符合的条件。
```

```vb
Option Explicit
'鸡兔同笼 VB Code by firendless
Private Sub cmdASK_Click()
Dim lngNumA As Long, lngNumB As Long, lngTmp As Long
lngNumA = 0: lngNumB = 0
Do
lngNumB = InputIntegerNumBox("请输入鸡和兔的下肢之和", "Fir Says:")
If lngNumB Mod 2 Then lngNumB = 0
Loop Until lngNumB > 0
Do
lngNumA = InputIntegerNumBox("请输入鸡和兔的脑袋之和", "Fir Says:")
If lngNumA * 2 >= lngNumB Then lngNumA = 0
Loop Until lngNumA > 0
Dim lngIndex As Long
For lngIndex = 0 To lngNumA
lngTmp = lngNumA - lngIndex
If lngNumA <> 0 Then
If 4 * lngIndex + 2 * lngTmp = lngNumB Then
MsgBox "算法A：" & vbNewLine & "鸡" & lngIndex & vbNewLine & "兔" & lngTmp
End If
End If
Next
lngTmp = (4 * lngNumA - lngNumB) / 2
lngIndex = (lngNumB - 2 * lngNumA) / 2
MsgBox "算法B：" & vbNewLine & "鸡" & lngIndex & vbNewLine & "兔" & lngTmp
End Sub
Private Function InputIntegerNumBox(Optional strLabel As String = "请输入一个数字", Optional strCaption As String = "程序提示") As Long
On Error Resume Next
Dim strTmp As String, bPoint As Byte
Do
If strTmp = "Err" Then strLabel = "Input A Integer,Plz."
strTmp = InputBox(strLabel, strCaption)
If InStr(strTmp, "-") Then strTmp = "Err"
bPoint = InStr(strTmp, ".")
If bPoint Then
bPoint = bPoint + 1
If InStr(bPoint, strTmp, ".") Then
strTmp = "Err"
Else
If Val(Mid$(strTmp, bPoint)) <> 0 Then strTmp = "Err"
End If
End If
Loop Until IsNumeric(strTmp) = True
InputIntegerNumBox = Val(strTmp)
End Function
```

