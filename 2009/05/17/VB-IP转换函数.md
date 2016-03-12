# [VB]IP转换函数

写了一个IP转换函数，突然有个2想法，每个函数都必须要而强的话，那么算法函数就不必去检查传入数值的有效性。 这个工作应该由其他函数来做；可读性可以用注释表示，算法直接最简就可以了，例如下面的例子中的求和。

`sum=p+p+p`

`sum=p *3`

```vb
Option Explicit

Private Const TestIP01 As String = "211.136.108.171"

Private Const TestIP02 As String = "3548933291"

Public Function IPConvert(strIP As String, bMode As Byte) As String

On Error GoTo hErr

Dim bIndex As Byte, bTmp As Byte, dblSum As Double, strArr() As String, bArr(3) As Byte, oTmp As Currency

Select Case bMode

Case 1

strArr = Split(strIP, ".", -1, vbTextCompare)
bArr(3) = CByte(strArr(0))
bArr(2) = CByte(strArr(1))
bArr(1) = CByte(strArr(2))
bArr(0) = CByte(strArr(3))

'(bArr(3) Mod 256) * (256 ^ 3) + (bArr(2) Mod 256) * (256 ^ 2) + (bArr(1) Mod 256) * (256 ^ 1) + (bArr(0) Mod 256) * (256 ^ 0)
dblSum = bArr(3) * 256 ^ 3 + bArr(2) * 256 ^ 2 + bArr(1) * 256 + bArr(0) * 1

IPConvert = dblSum

Case 2

dblSum = CDbl(strIP)
bTmp = 0

For bIndex = 0 To 3

bTmp = 3 - bIndex
oTmp = Int(dblSum / 256 ^ bTmp)
bArr(bIndex) = CByte(oTmp)
dblSum = dblSum - (bArr(bIndex) * 256 ^ bTmp)

Next

IPConvert = CStr(bArr(0)) & "." & CStr(bArr(1)) & "." & CStr(bArr(2)) & "." & CStr(bArr(3)) & "."

End Select

Exit Function

hErr:
IPConvert = ""
End Function

Private Sub cmdCommand1_Click()

MsgBox IPConvert(TestIP01, 1)
MsgBox IPConvert(TestIP02, 2)

End Sub
```

