# [VB]从变量来看初学容易犯的错误

**声明：本文作者也很菜，水平有限，如果有错误，欢迎指出。**

首先不强制声明变量的情况下，我们运行一下，看看会输出什么东西。
下面这段是一个简单的循环，是让变量i在窗体载入的时候进入一个1到10的循环，
最后输出结果和数据类型。

```vb
Private Sub Form_Load()
'苏洋注释：建立一个空循环来验证变量是否可用
For i = 1 To 10
Next
'输出结果
Debug.Print &amp;quot;数据数值:&amp;quot; &amp;amp; i, &amp;quot;数据类型:&amp;quot; &amp;amp; VarType(i)
End Sub
```


运行了一下，结果很明晰，“数据数值:11 数据类型:2”，查表可知，2是integer。
或许你会说，没什么，很正常的，自动化类型么，我的实验目的是什么~
如果你这么问了，说明你对优化程序细节还是没有建立初步的概念。
上面的循环范围在1~10完全可以使用byte类型来做的，如果使用integer或者long也不是不行，不过是增加了内存开支。
如果你有兴趣，可以阅读全文。

<!-- more -->

如果我们这样子写会有什么问题呢？


```vb
Private Sub Form_Load()
'苏洋注释：建立一个空循环来验证变量是否可用
For i = 1 To 10
Dim i As Byte
Next
'输出结果
Debug.Print "数据数值: "  & i,  "数据类型: "  & VarType(i)
End Sub
```

如果没猜错，你的编译器也会报警，说当前范围内声明重复。
为什么呢，其实很简单，因为编译器在编译器会检查for后面跟随的变量是否声明，
没有的话会生成一个全局变量，这句话是网上广为流传的。[很占内存的类型]
看看下面这个例子或许你会有所启示。


```vb
Private Sub Form_Load()
Dim i As Byte: i = 0
Debug.Print  "数据数值： "  & i,  "数据类型： "  & VarType(i)
Dim j As Byte
Call MySub
End Sub

Private Sub MySub()
Debug.Print  "数据数值： "  & j,  "数据类型： "  & VarType(j)
End Sub
```

运行结果是：“数据数值：0 数据类型：17
数据数值： 数据类型：0”
查表可知，数据类型0为empty(未初始化)，而并非自动初始化为0哦。如果我们再代码首部加一个Option Explicit，
则会出现变量j未定义。是不是印证了上面的话？"没有的话会生成一个全局变量"别着急，继续往下看。


```vb
Private Sub Form_Load()
Dim i As Byte: i = 0
Debug.Print  "数据数值： "  & i,  "数据类型： "  & VarType(i)
Dim j As Byte
j = 1
Call MySub
End Sub

Private Sub MySub()
Debug.Print  "数据数值： "  & j,  "数据类型： "  & VarType(j)
End Sub
```

再修改一下，看看运行结果“数据数值：0 数据类型：17
数据数值： 数据类型：0”
似乎是一样，其实是不一样的，我们在一个过程中赋值后并不能影响到另外一个过程中的数值，这个说明什么呢？
如果两个过程中都有一个变量会各自生产局部变量，而不是广为流传的全局变量。
也可以理解为如果某作用域中没有该变量程序会继续向上查看是否存在变量限制，在查找的过程会使用类似贪婪算法而不是包袱算法。
或许你会说是我的dim j的限制作用，好吧，我去掉那句，我们继续运行


```vb
Private Sub Form_Load()
Dim i As Byte: i = 0
Debug.Print  "笔菔担?"  & i,  "数据类型： "  & VarType(i)

j = 1
Call MySub
End Sub

Private Sub MySub()
Debug.Print  "数据数值： "  & j,  "数据类型： "  & VarType(j)
End Sub
```

这样子是不是似乎会生成一个全局变量j呢，并且会自动初始化为1.
运行一下结果是“数据数值：0 数据类型：17
数据数值： 数据类型：0”
似乎网上的经典传言被消灭...老是否决别人也不是很好，所以接下来，我们验证一个东西。


```vb
Private Sub Form_Load()
Dim j As Variant
Debug.Print  "数据数值： "  & j,  "数据类型： "  & VarType(j)
End Sub
```

返回结果为“数据数值： 数据类型：0”
结合之前的数据看到的数据类型的结果，“dim a 相当于 dim a as varient”是正确的，
我相信很多初学者都会有过崇拜权威的错误吧，看到那个老鸟说默认全局变量就全局了，或者默认变体就变体了。
为什么不自己试验一下呢，只需要简单的几句话而已。
最后补充一下
sub(子过程)的全称为：Subroutine
dim(声明)的全称为：Dimension
或许看到全称后，你又会有所思考。


