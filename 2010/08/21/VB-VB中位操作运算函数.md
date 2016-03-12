# [VB]VB中位操作运算函数

很久没有用VB写东西了今天突然想写个挂机程序,想起VB不能直接进行位操作,特到水木清华扒到一篇。

```vb
发信人: hermit (阿修罗～相拥我爱), 信区: VisualBasic       
标  题:  VB中位操作运算函数【移位指令】
发信站: BBS 水木清华站 (Sat Jun  1 12:40:23 2002)

'Module: BitPlus.Bas
'Code By Hermit @ SMTH , Jun. 1st,2000
'Email: mailtocw@sohu.com
'May these functions will help you, and
'Please keep this header if you use my code,thanks!
'提供在VB下进行位运算的函数
'SHL 逻辑左移  SHR  逻辑右移
'SAL 算术左移  SAR  算术右移
'ROL 循环左移  ROR  循环右移
'RCL 带进位循环左移  RCR  带进位循环右移
'Bin 将给定的数据转化成2进制字符串
'使用方法
'SHL SHR SAL SAR ROL ROR 基本类似，以SHL为例说明
'可以移位的变量类型，字节(Byte)，整数(Integer)，长整数(Long)
'返回值 True 移位成功， False 移位失败,当对非上述类型进行移位是会返回False
'Num 传引用变量，要移位的数据，程序会改写Num的值为运算后结果
'iCL 传值变量，要移位的次数，缺省值移位1次
'例 Dim A As Integer
'   A = &H10
'如 SHL A    则移位后 A = &H20
'如 SHL A,2  则移位后 A = &H40
'如 SHL A,4  则移位后 A = &H00
'RCR与RCL类似，以RCL为例说明
'这里需要多给定一个参数，即第一次移位时的进位值iCF
'Bin举例
'A = &H1
'如 A 为字节，则 Bin(A) 返回值为 "00000001"
'如 A 为整数，则 Bin(A) 返回值为 "0000000000000001"
'如 A 为长整数，则 Bin(A) 返回值为 "00000000000000000000000000000001"
'如果传入参数非上述类型时，返回值为 ""
'更详细的信息，请参考相关汇编书籍
'逻辑左移
Public Function SHL(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
Dim i As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    iMask = 0
    If (Num And &H4000) <> 0 Then iMask = &H8000
    Num = (Num And &H3FFF) * 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    lMask = 0
    If (Num And &H40000000) <> 0 Then lMask = &H80000000
    Num = (Num And &H3FFFFFFF) * 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    bMask = 0
    If (Num And &H40) <> 0 Then bMask = &H80
    Num = (Num And &H3F) * 2 Or bMask
  Next
Case Else
  SHL = False
  Exit Function
End Select
SHL = True
End Function
'逻辑右移
Public Function SHR(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
Dim i As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    iMask = 0
    If (Num And &H8000) <> 0 Then iMask = &H4000
    Num = (Num And &H7FFF) \ 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    lMask = 0
    If (Num And &H80000000) <> 0 Then lMask = &H40000000
    Num = (Num And &H7FFFFFFF) \ 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    bMask = 0
    If (Num And &H80) <> 0 Then bMask = &H40
    Num = (Num And &H7F) \ 2 Or bMask
  Next
Case Else
  SHR = False
  Exit Function
End Select
SHR = True
End Function
'算术左移
Public Function SAL(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
SAL = SHL(Num, iCL)
End Function
'算术右移
Public Function SAR(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
Dim i As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    iMask = 0
    If (Num And &H8000) <> 0 Then iMask = &HC000
    Num = (Num And &H7FFF) \ 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    If (Num And &H80000000) <> 0 Then lMask = &HC0000000
    Num = (Num And &H7FFFFFFF) \ 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    If (Num And &H80) <> 0 Then bMask = &HC0
    Num = (Num And &H7F) \ 2 Or bMask
  Next
Case Else
  SAR = False
  Exit Function
End Select
SAR = True
End Function
'循环左移
Public Function ROL(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
Dim i As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    iMask = 0
    If (Num And &H4000) <> 0 Then iMask = &H8000
    If (Num And &H8000) <> 0 Then iMask = iMask Or &H1
    Num = (Num And &H3FFF) * 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    lMask = 0
    If (Num And &H40000000) <> 0 Then lMask = &H80000000
    If (Num And &H80000000) <> 0 Then lMask = lMask Or &H1
    Num = (Num And &H3FFFFFFF) * 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    bMask = 0
    If (Num And &H40) <> 0 Then bMask = &H80
    If (Num And &H80) <> 0 Then bMask = bMask Or &H1
    Num = (Num And &H3F) * 2 Or bMask
  Next
Case Else
  ROL = False
  Exit Function
End Select
ROL = True
End Function
'循环右移
Public Function ROR(ByRef Num As Variant, Optional ByVal iCL As Byte = 1) As
 Boolean
Dim i As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    iMask = 0
    If (Num And &H8000) <> 0 Then iMask = &H4000
    If (Num And &H1) <> 0 Then iMask = iMask Or &H8000
    Num = (Num And &H7FFF) \ 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    lMask = 0
    If (Num And &H80000000) <> 0 Then lMask = &H40000000
    If (Num And &H1) <> 0 Then lMask = lMask Or &H80000000
    Num = (Num And &H7FFFFFFF) \ 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    bMask = 0
    If (Num And &H80) <> 0 Then bMask = &H40
    If (Num And &H1) <> 0 Then bMask = bMask Or &H80
    Num = (Num And &H7F) \ 2 Or bMask
  Next
Case Else
  ROR = False
  Exit Function
End Select
ROR = True
End Function
'带进位循环左移
Public Function RCL(ByRef Num As Variant, Optional ByVal iCL As Byte = 1, Op
tional ByVal iCf As Byte = 0) As Boolean
Dim i As Byte, CF As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
CF = iCf
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    If CF = 0 Then
       iMask = 0
    Else
       iMask = 1
    End If
    If (Num And &H4000) <> 0 Then iMask = iMask Or &H8000
    If (Num And &H8000) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H3FFF) * 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    If CF = 0 Then
       lMask = 0
    Else
       lMask = 1
    End If
    If (Num And &H40000000) <> 0 Then lMask = lMask Or &H80000000
    If (Num And &H80000000) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H3FFFFFFF) * 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    If CF = 0 Then
       bMask = 0
    Else
       bMask = 1
    End If
    If (Num And &H40) <> 0 Then bMask = bMask Or &H80
    If (Num And &H80) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H3F) * 2 Or bMask
  Next
Case Else
  RCL = False
  Exit Function
End Select
RCL = True
End Function
'带进位循环右移
Public Function RCR(ByRef Num As Variant, Optional ByVal iCL As Byte = 1, Op
tional ByVal iCf As Byte = 0) As Boolean
Dim i As Byte, CF As Byte
Dim bMask As Byte, iMask As Integer, lMask As Long
CF = iCf
Select Case VarType(Num)
Case 2 '16 bits
  For i = 1 To iCL
    If CF = 1 Then
       iMask = &H8000
    Else
       iMask = 0
    End If
    If (Num And &H8000) <> 0 Then iMask = iMask Or &H4000
    If (Num And &H1) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H7FFF) \ 2 Or iMask
  Next
Case 3 '32 bits
  For i = 1 To iCL
    If CF = 1 Then
       lMask = &H80000000
    Else
       lMask = 0
    End If
    If (Num And &H80000000) <> 0 Then lMask = lMask Or &H40000000
    If (Num And &H1) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H7FFFFFFF) \ 2 Or lMask
  Next
Case 17 '8 bits
  For i = 1 To iCL
    If CF = 1 Then
       bMask = &H80
    Else
       bMask = 0
    End If
    If (Num And &H80) <> 0 Then bMask = bMask Or &H40
    If (Num And &H1) <> 0 Then
       CF = 1
    Else
       CF = 0
    End If
    Num = (Num And &H7F) \ 2 Or bMask
  Next
Case Else
  RCR = False
  Exit Function
End Select
RCR = True
End Function
'将数值转化为二进制字符串
Public Function Bin(ByVal Num As Variant) As String
Dim tmpStr As String
Dim iMask As Long
Dim iCf As Byte, iMax As Byte
Select Case VarType(Num)
Case 2: iMax = 15 'Integer 16 bits
Case 3: iMax = 31 'Long 32 bits
Case 17: iMax = 7 'Byte 8  bits
Case Else
  Bin = ""
  Exit Function
End Select
iMask = 1
If iMask And Num Then
   tmpStr = "1"
Else
   tmpStr = "0"
End If
For iCf = 1 To iMax
   If iCf = 31 Then
      If Num > 0 Then
         tmpStr = "0" + tmpStr
      Else
         tmpStr = "1" + tmpStr
      End If
      Exit For
   End If
   iMask = iMask * 2
   If iMask And Num Then
      tmpStr = "1" + tmpStr
   Else
      tmpStr = "0" + tmpStr
   End If
Next
Bin = tmpStr
End Function

```

