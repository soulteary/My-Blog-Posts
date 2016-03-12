# [vb]算数精度

```vb
Dim i As Integer  
Dim Sum As Single  

For i = 0 To 100   
  Sum = Sum + 0.01   
Next i   
Text1.Text = Sum
```

和等于 1.01，但在 Text1.Text 显示 1.009999

```vb
Dim i As Integer  
Dim x As Single  
Dim y As Single  

For i = 0 To 1000   
  x = x + 0.01   
Next i   
y = 10.082 - 0.072   

If (x = y) Then Text3.Text = "相等"  
Text4.Text = x   
Text5.Text = y
```

最后不会显示"相等!"因为 10.01 不能完全表示 可以修改成如下：

```vb
Dim i As Integer  
Dim x As Single  
Dim y As Single  

For i = 0 To 1000   
  x = x + 0.01   
Next i   
y = 10.082 - 0.072   

If ((x - y) < 0.01 And (x - y) > -0.01) Then  
  Text3.Text = "Equal!"  
End If  
Text4.Text = x   
Text5.Text = y   
```

此外需要注意，操作长整形数学的时候 Visual Basic 建议存储中间值。

例如计算 CInt(4.555 * 100)  455.5 是CInt() 函数中临时存储。
如果在两个的步骤中分为计算 x = 4.555 * 100 和 CInt(x)，避免此内部存储，避免浮点舍入错误。

下面是其他常见的浮点错误： 
舍入错误-此错误导致，在所有位二进制数不能在计算中使用。
溢出和下溢-太大或太小，由该数据类型结果时将发生此错误。
部门通过极少数的这可以触发"被零除"错误或可能会产生不正确的结果。
注意 ： 这不是特别是 Visual Basic 问题。 则存在与 C、 FORTRAN 或存储浮点数字根据为 IEEE 标准的任何其他语言相同的问题。


