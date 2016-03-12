# [vb]比较日期

日期比较方法 看到这个文章你一定会觉得很诧异，别诧异，看看就知道了。 

使用运算符会造成误差，解决办法是使用函数，该函数有较低的容差。

```vb
Option Explicit   

Private Sub Form_Load()   

    Dim Date1 As Date  

    Dim Date2 As Date  

    Dim Date3 As Date  

    Dim Date4 As Date  

    Me.Width = 6500   
    Me.Height = 3000   
    Me.Show   
    Date1 = #10/21/1998 8:00:00 AM#   
    Date2 = #10/21/1998 8:20:00 AM#   
    Date3 = DateAdd("n", 20#, Date1)   
    Date4 = Date1 + TimeSerial(0, 20, 0)   
    Print "The results are visually identical..."  
    Print   
    Print "Date2 = "; Date2   
    Print "Date3 = "; Date3   
    Print "Date4 = "; Date4   
    Print   
    Print "but the actual values are not"  
    Print   
    Print Tab(20), "=", "DateDiff", "Actual Difference"  
    Print "Date2 = Date3?", Date2 = Date3,   
    Print DateDiff("s", Date2, Date3), Date2 - Date3   
    Print "Date2 = Date4?", Date2 = Date4,   
    Print DateDiff("s", Date2, Date4), Date2 - Date4   
    Print "Date3 = Date4?", Date3 = Date4,   
    Print DateDiff("s", Date3, Date4), Date3 - Date4   
End Sub

```

