# [我的总结]关于ROUND函数的BUG

ROUND函数在很多语言中都存在，它的“bug”也威名远播... 关于它的舍入问题，很多人都总结过吧，四舍六入五成双。 文章包含代码以及一些引用资料。

<!-- more -->

> 以下资料来自[CSDN](http://www.promiseforever.com/redirect.php?url=http://topic.csdn.net/u/20080111/15/a8c274c3-901b-433c-9f7f-43f306c29ff9.html&key=0e404da57ab7416a52073d41674a7fb3) 以[及黄海blog](http://www.promiseforever.com/redirect.php?url=http://www.msaccessonline.com/post/63.html&key=5a9f5f974156d3b0aa5e0cc37631860f)

ROUND函数在很多语言中都存在，它的“bug”也威名远播... 关于它的舍入问题，很多人都总结过吧，四舍六入五成双。 文章包含代码以及一些引用资料，如果感兴趣，可以选择阅读全文。

> 以下资料来自[CSDN](http://www.promiseforever.com/redirect.php?url=http://topic.csdn.net/u/20080111/15/a8c274c3-901b-433c-9f7f-43f306c29ff9.html&key=0e404da57ab7416a52073d41674a7fb3) 以[及黄海blog](http://www.promiseforever.com/redirect.php?url=http://www.msaccessonline.com/post/63.html&key=5a9f5f974156d3b0aa5e0cc37631860f)

其实财务函数的它主要是进行一种平衡算法。 正如一位朋友所言 ， “VBA中的Round函数不是Bug，而只是一种特殊算法（银行业算法）。 简单地说就是向最近的偶数位舍入。 如：Round(3.375,2)==Round(3.385,2)==3.38” 下面的这个函数（算术四舍五入平衡算法）考虑了负数的处理改编自微软文档，可以用于财务系统开发：

```vb
Function SARound(ByVal X As Currency, Optional ByVal Factor As Long = 2) As Currency
SARound = Fix(X * 10 ^ Factor + Sgn(X) * 0.5) / 10 ^ Factor
End Function
```

如果把Fix变为Int就成为不平衡算法。 MS文档原文： http://support.microsoft.com/default.aspx?scid=kb;en-us;196652

```vb
Public Function RoundToLarger(dblInput As Double, intDecimals As Integer) As Double
Dim strFormatString As String
If dblInput <> 0 Then
strFormatString = "#." & String(intDecimals, "#")
RoundToLarger = Format(dblInput, strFormatString)
Else
RoundToLarger = 0
End If
End Function

补充一种方法:
'自定义自四舍五入函数
'解决ACCESS97以下版本不支持Round函数
'解决Round"有名"的四舍六入现象
'参数: Number , 要进入四舍五入的数值
'参数:N,要保留的小数位数,不足时以0补上

'用法:
'Print myRound(1.4367, 2)
'1.44
Function myRound(Number As Double, N As Integer) As String
myRound = Format(Int(Number * (10 ^ N) + 0.5) / (10 ^ N), "0." & String(N, "0"))
End Function
```

说到原理，这里有一个C的实现

```c
private static unsafe double InternalRound(double value, int digits, MidpointRounding mode)
{
if (Abs(value) < doubleRoundLimit)
{
double num = roundPower10Double[digits];
value *= num;
if (mode == MidpointRounding.AwayFromZero)
{
double num2 = SplitFractionDouble(&value);
if (Abs(num2) >= 0.5)
{
value += Sign(num2);
}
}
else
{
value = Round(value);
}
value /= num;
}
return value;
}
```

还有一位指出在VS中使用C#,调用数值返回ROUND不准确，和没有确切指出数值类型有关

```c
class Program
{
static void Main(string[] args)
{
Console.WriteLine("Math.Round(4.00095, 4):{0}", Math.Round(4.00095M, 4));
Console.WriteLine("Math.Round(4.00195, 4):{0}", Math.Round(4.00195M, 4));
Console.WriteLine("Math.Round(7.00095, 4):{0}", Math.Round(7.00095M, 4));
Console.WriteLine("Math.Round(7.00195, 4):{0}", Math.Round(7.00195M, 4));
Console.WriteLine("Math.Round(5.00095, 4):{0}", Math.Round(5.00095M, 4));
Console.WriteLine("Math.Round(5.00195, 4):{0}", Math.Round(5.00195M, 4));
Console.WriteLine("Math.Round(5.0095, 3):{0}", Math.Round(5.0095M, 3));

Console.WriteLine("Math.Round(5.0295, 3):{0}", Math.Round(5.0295M, 3));
Console.WriteLine("Math.Round(5.0395, 3):{0}", Math.Round(5.0395M, 3));

Console.WriteLine("按任意鍵退出...");
Console.ReadKey();
}
}
```

之所以和你的結果不一樣，是因為你的代碼中直接用4.00095來表示，這樣C#會認為是浮點數double類型，而浮點數的表示往往會不準確，但是如果加個M後綴，則表示為Decimal類型，因此會準確一些。 为了防止文章失效，我把csdn和msdn上的文档保存为文本..

下载地址： [download id="14"]

