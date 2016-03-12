# [vb]获得程序某个过程运行时间

想知道你写的代码的某个函数或者过程运行完毕花了多长时间么，看看这篇文章吧。

例子中是随即生成100万个数字，然后进行直接排序。

```vb
Option Explicit   
  
Private Sub Main()   
       
    Dim tmr As Single  
  
    tmr = Timer   
       
    Dim Value(999999) As Byte  
      
    Dim i             As Single  
      
    Randomize   
      
    For i = 0 To 999999   
      
        Value(i) = Int(Rnd * 100)   
      
    Next i   
      
    Dim intMax As Integer  
      
    intMax = 0   
      
    For i = 0 To 999999   
      
        If Value(intMax) < Value(i) Then intMax = i   
      
        i = i + 1   
      
    Next i   
       
    MsgBox "运行时间：" & Timer - tmr & "  ms" & vbNewLine & "最大值：" & Value(intMax) & vbNewLine & "最大值序号" & intMax + 1   
  
End Sub
```


