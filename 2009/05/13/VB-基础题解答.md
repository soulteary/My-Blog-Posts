# [VB]基础题解答

```text
百度知道上的三道题,最后一道有参考知道网友的回复[我居然想一位一位的拆开,mY God.]
1、从3个红球、5个白球、6个黑球中任意取出8个球，且其中必须有白球，编程输出所有可能的方案。
2、求一个整数任意次方的最后三位数，即求x^y的后三位。
3、输出1-100之间，每位数的乘积大于每位数的和的整数。
```

```vb
Option Explicit

Private Sub Form_Load()

'****************************************************
'1、 从3个红球、5个白球、6个黑球中任意取出8个球，且其中必须有白球，编程输出所有可能的方案。

Dim bBallW As Byte, bBallR As Byte, bBallB As Byte
Dim bIndexW As Byte, bIndexR As Byte, bIndexB As Byte
Dim strTmp As String

bBallR = 3: bBallW = 5: bBallB = 6: strTmp = ""

For bIndexW = 1 To bBallW

For bIndexR = 0 To bBallR

For bIndexB = 0 To bBallB

If bIndexW + bIndexB + bIndexR = 8 Then strTmp = strTmp & "白球：" & bIndexW & "黑球：" & bIndexB & "红球：" & bIndexR & vbNewLine

Next

Next

Next

MsgBox strTmp

'****************************************************
'2、 求一个整数任意次方的最后三位数，即求x^y的后三位。

Dim strRtn As String
Dim lngNum As Long, lngLv As Long

lngNum = InputIntegerNumBox
lngLv = InputIntegerNumBox("请输入该数字的指数")
strRtn = Right(CStr(lngNum ^ lngLv), 3)

MsgBox strRtn

'****************************************************
'3、 输出1-100之间，每位数的乘积大于每位数的和的整数。

Dim bNum As Byte, bIndex As Byte, bNumA As Byte, bNumB As Byte

For bIndex = 10 To 99

bNumA = bIndex / 10
bNumB = bIndex Mod 10

If bNumA * bNumB > bNumA + bNumB Then Debug.Print bIndex

Next

End Sub

Private Function InputIntegerNumBox(Optional strLabel As String = "请输入一个数字", Optional strCaption As String = "程序提示") As Long

On Error Resume Next

Dim strTmp As String

Do

If strTmp = "Err" Then strLabel = "请输入一个整数。"
strTmp = InputBox(strLabel, strCaption)
If InStr(1, strTmp, ".") Then strTmp = "Err"
Loop Until IsNumeric(strTmp) = True
InputIntegerNumBox = Val(strTmp)
End Function

```

