# [总结]浅谈程序的细节优化兼回keelii的留言

写这篇之前我先絮叨两句，印象中很久前有一个家伙给我Mail问我user-agent的相关，往来数封信后，问了我是干嘛的，苏洋同学自然是诚实的回复他，高三应届小P孩了，于是乎，便没了后文，真心的希望那个家伙是了解了user-agent以及的不同，而不是因为鄙夷我这个高中生还继续去Google过期的AD...

下面是一条旧博客的评论，今天看留言看到的，有点感触，于是写一篇日志。

> 大一的时候学过点vb不过现在都学B/S的东东了.每在意过vb! 本条评论由 keelii 于 2009, May 23, 3:14 PM 发表 #1

作为Code fans & Vb fans的我，对于这个问题我打算这么回复一下， 我不打算讨论VB重要与否，以及C时如何从B的影子中走出的，其实语言都一样的，只是一个工具而已，就和我们学英语日语一直，只是为了表达我们的想法。 唯一不同的是，语言的语法语义有别，比如dim在vb中是声明变量，而到了c就只能作为变量以及函数名称，printf在c中是标准输出函数，但是到了vb里就只能充当变量和函数名一样了。

另外无论是C/S还是B/S只要是编程语言就是可以实现的，区别只是在于实现的难易以及效率了，对于熟悉的人来说一句代码将会比菜菜的10句代码更有效果的哦~ 说了这句话不知道某只猫兄会不会来砸我... 上面的或许keelii会觉得有点空，那么我们来个实际的吧。 考验程序效率的方法无非是压力测试，我这里列举一个简单的，多叉树统计字符个数。basic对于字符操作一直是弱项，由于数据类型为ole_txt[需要初始化]，所以basic中处理字符是最痛苦的，但是看过本文或许你的观点会改变一些，当然使用内存操作以及内嵌其他语言的手段也可以大幅提高效率，我们暂且不谈。

> 孟子曰：“舜发于畎亩之中，傅说举于版筑之间，胶鬲举于鱼盐之中，管夷吾举于士，孙叔敖举于海，百里奚举于市。故天将降大任于是人也，必先苦其心志，劳其盘骨，儿其体肤，空管其身行，指乱其所为，所以动心忍性，曾益其所不能。人恒过，然后能改。困于心，衡于虑，而后作。徵于色，发于声，而后喻。入则无法家拂士，出则无敌国外患者，国恒亡。然后知生于忧患难与共，而死于安乐也。”

上面这段358字节的语段是孟子的告天下，我很喜欢~就用它来现身说法了。 下面有一段程序，可以说写的比较草，但是实现了功能。 http://zhidao.baidu.com/question/28498709.html

```vb
'要准确地统计字数，可逐一将字符串转换为ASCII码，依据其值判断是为中文字符还是英文字符。0——127之间的为大小写字母及数字、半角标点符号、回车、换行等，中文字符的ASCII值则不在0——127之列了。这样，纯汉字的字数统计是很容易的，倒是英文的统计复杂，因为英文统计应以单词为单位，而要判断是否为单词并不是简单的事。我们可以这么处理：如果被检测的字符为大小写字母，则判断其后面的字符是否为一个单词的标志(如空格、标点符号、回车符等），如是，则判断为一个单词。
'以下代码能较准确地统计出TextBox控件中的中、英文字数和数字字符数，并将全部字节数(含各种控制符如回车等)也统计出来。适用于中英文编排环境。

'注释：窗体级声明
Dim c As Long, e_word As Long 注释：中英文字数
Dim Num As Long, s As Long 注释：数字及全部字符数

'注释：统计——
Private Sub Command1_Click()

Dim Str As String 注释：总字符
Dim k As Long 注释：计数器
Dim tmpStr As String 注释：逐一检测的字符

c = 0: e_word = 0: Num = 0: s = 0 注释：清空变量
Str = Text1.Text & " " 注释：加一空格便于意外时计算最后一个字符
For k = 1 To Len(Str) - 1
tmpStr = Mid$(Str, k, 1)

If Asc(tmpStr) >= 65 And Asc(tmpStr) <= 90 Then 注释：小写字母
If Asc(Mid$(Str, k + 1, 1)) <= 64 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) > 90 And Asc(Mid$(Str, k + 1, 1)) < 97 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) > 122 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) = 39 Or Asc(Mid$(Str, k + 1, 1)) = 45 Then e_word = e_word - 1 注释：是符号注释：或-时
ElseIf Asc(tmpStr) >= 97 And Asc(tmpStr) <= 122 Then 注释：大写字母
If Asc(Mid$(Str, k + 1, 1)) < 65 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) > 90 And Asc(Mid$(Str, k + 1, 1)) < 97 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) > 122 Then e_word = e_word + 1
If Asc(Mid$(Str, k + 1, 1)) = 39 Or Asc(Mid$(Str, k + 1, 1)) = 45 Then e_word = e_word - 1 注释：是符号注释：或-时
ElseIf Asc(tmpStr) >= 48 And Asc(tmpStr) <= 57 Then 注释：阿拉伯数字数字
If Asc(Mid$(Str, k + 1, 1)) < 48 Or Asc(Mid$(Str, k + 1, 1)) > 57 Then Num = Num + 1
ElseIf Asc(tmpStr) > 127 Or Asc(tmpStr) < 0 Then 注释：中文字符
c = c + 1
End If
Next

s = LenB(StrConv(Text1.Text, vbFromUnicode)) 注释：全部字符

MsgBox "本文共有：" & vbCrLf & vbCrLf & "汉字字数: " & c & _
" 个 (含全角标点)" & vbCrLf & "英文单词: " & e_word & " 个 (不含半角标点)" & vbCrLf & _
"数字: " & Num & " 个" & vbCrLf & "全部字节: " & s & " 个", vbInformation, "字数统计"

End Sub 
```

再次说明上面这个是随手搜索的百度得到的一个答案..不是刻意找麻烦的哦~ 在过程前加一句

```vb
Dim tim As Single: tim = Timer
```

过程结束前的对话框前加一句

```vb
MsgBox "处理完成，消耗时间: " & Format(Timer - tim, "@@@@@@") & " ms"
```

然后叫它处理一下孟子的告天下... 弹出结果之后又弹出了一个对话框，显示0.015625ms... 我的笔记本由于长期开始或许影响了运算速度，请作者别介意哈~ 但是苏洋同学还是觉得不爽了...程序就是要节约生命的~于是乎，修改一下吧。

```vb
Option Explicit

Dim lngZH As Long, lngEN As Long, lngNum As Long, lngTotal As Long

Private Sub Command1_Click()

lngZH = 0: lngEN = 0: lngNum = 0: lngTotal = 0

Dim tim As Single: tim = Timer

Dim strTmp As String, strChar As String, strNext As String

Dim lngLength As Long, lngIndex As Long, lngChar As Long

lngLength = 0: lngIndex = 0

strTmp = Text1.Text & " "

lngLength = Len(strTmp) - 1

For lngIndex = 1 To lngLength
strChar = Mid$(strTmp, lngIndex, 1)
lngChar = Asc(strChar)

If lngChar >= 65 And lngChar <= 90 Then
strNext = Mid$(strTmp, lngIndex + 1, 1)
lngChar = Asc(strNext)

If lngChar <= 64 Then lngEN = lngEN + 1
If lngChar > 90 And lngChar < 97 Then lngEN = lngEN + 1
If lngChar > 122 Then lngEN = lngEN + 1
If lngChar = 39 Or lngChar = 45 Then lngEN = lngEN - 1
ElseIf lngChar >= 97 And lngChar <= 122 Then
strNext = Mid$(strTmp, lngIndex + 1, 1)
lngChar = Asc(strNext)

If lngChar < 65 Then lngEN = lngEN + 1
If lngChar > 90 And lngChar < 97 Then lngEN = lngEN + 1
If lngChar > 122 Then lngEN = lngEN + 1
If lngChar = 39 Or lngChar = 45 Then lngEN = lngEN - 1
ElseIf lngChar >= 48 And lngChar <= 57 Then
strNext = Mid$(strTmp, lngIndex + 1, 1)
lngChar = Asc(strNext)
If lngChar < 48 Or lngChar > 57 Then lngNum = lngNum + 1
ElseIf lngChar > 127 Or lngChar < 0 Then
lngZH = lngZH + 1
End If

Next

lngTotal = LenB(StrConv(Text1.Text, vbFromUnicode))

MsgBox "处理完成，消耗时间: " & Format(Timer - tim, "@@@@@@") & " ms"

MsgBox "本文共有：" & vbCrLf & vbCrLf & "汉字字数: " & lngZH & " 个 (含全角标点)" & vbCrLf & "英文单词: " & lngEN & " 个 (不含半角标点)" & vbCrLf & "数字: " & lngNum & " 个" & vbCrLf & "全部字节: " & lngTotal & " 个", vbInformation, "字数统计"

End Sub
```

修改完后出现一个小问题...告天下对于刚刚的程序来说数据量太小了，
于是我换成了"暮光之城 破晓.txt" 大约是292kb，运行后弹出0.109375ms。
还凑乎吧~数据量提高了750倍后仅仅多用了0.08ms多,如果把结果除以750
等量的话，那么我们的效率高了多少呢~我的这个修改也是很简单的，还是一样的算法，况且这个多叉树已经写的很完善了，
我没有进行修改，修改的仅仅是细节而已。
有时候我们不在意的细节，会使得结果造成很大的区别。
无论是vb，.net，还是c#，css等，道理是相同的。
当然术业有专攻，比如处理数据，还是c来的快，
遍历网页元素，我们绝对会选择css而不是js。
keelii同学，这个只是一个浅显的例子，
但是可以说明一个简单的道理，学校里学到的永远是不够的，这句话已经有很多人和我说过了，
我现在也送给你，在学习的路上，我们要学会慎独。(俺还没有专门的学过计算机的说)
有时候看起来越是简单的东西，细节方面越是需要我们去注意。

这篇文章由苏洋乱笔写下，如有冲突，还请海涵。
如果有错误的话，欢迎指正。

