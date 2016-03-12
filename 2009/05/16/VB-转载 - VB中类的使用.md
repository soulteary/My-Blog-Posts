# [VB]转载 - VB中类的使用

贴一篇从网上看到的文章吧，关于VB中类的使用。2000年左右的老文了。文章很长，但是阅后无论你对类了解多或少，都会获得一些新的视角。

<!--more-->

作者：黄河

声明

一、这一系列的贴子完全是我的原创文章。是我应腾讯讨论组VB版中的一些朋友的要求写的。
二、从技术上讲，我不是高手，所以请大家在读这系列贴子的时候要当心，因为我可能因为我知识的局限性而写出错误的观点——尽管我尽力避免犯错。
三、欢迎大家收集或转贴这一系列的贴子，但请您一定要把“作者：腾讯讨论组VB版黄河”这几个字带上。
四、对于这一系列的贴子，我自己已经有一个提刚——尽管这些贴子并没全部写完——但如果您有什么地方需要我讲得更详细，请到腾讯讨论组VB版发贴说明。
五、我的英文不好，如果你发现我的贴子中的英文有用词不当的地方，请原谅。至于中文水平，我觉得对自己的信心还是有一点的，如果因为我的中文水平给您带来了什么麻烦，那属于意外，对不起了。

第一天：类的概念

当您第一次看到“类”这个概念时，可能会觉得摸不着头脑。我们先看一点我们熟悉的东西：
在VB控件工具箱中的CommandButton，这是VB中的按钮控件，是我们在VB程序中经常用到的基本控件。我们在学习VB的类之前，单纯地就把它看成一个控件，其实，用类的观点，它就是一个类。我们知道，所有CommandButton都有相同的属性选项，尽管对于画到窗体上去的每个按钮，这些属性的值可能不同；它们也都有相同的事件，尽管我们对画到窗体上的每个按钮的这些事件地处理可能不同；它们也都有不同的方法，尽管我们调用每个画到窗体上的按钮的方法的目的不同。但，所有的窗体中的按钮都是CommandButton这一类控件。
我们新建一个窗体，从控件工具箱中选中CommandButton并画到窗体上，这时，窗体上就有了一个我们非常熟悉的Command1这个按钮。我们看看Command1这个东西，按照类的观点，它现在不能再叫做一个CommandButton控件（类）了，而叫做一个按钮，是CommandButton这个控件（类）的实例。所谓实例就是被具体化的类的一个形态，它有自己的属性，如高度和宽度，有自己的位置和大小，有自己的Caption和Name；它有自己的方法，如Move，当我们用Command1.Move这个方法时，谁都知道，只会移动Command1;它有自己的事件，如Click事件，当我们对Command1_Click进行代码编写后，只有Command1会调用我们对这个事件编写的代码。每当我们在这个窗体上新画一个CommandButton，就新产生一个CommandButton的实例，每个新产生的按钮，都有自己的个性，但它们不会有CommandButton这个类所包含的属性项目以外的选择，比如Command1绝对不会有Max属性。
我们再看第二个例子，这个例子我想跳出VB的范围，回到生活中来。比如我们常说，程序员是一类人，在这里，我们把程序员这类人就叫做一个类。这个类有一些属性，比如Name(姓名)、EmpolderTool（使用的开发工具）；这个类有一些方法，比如Empolder（开发）、Study（学习）；也同样有一些事件，比如EmpolderSucceed（开发成功）、EmpolderLost(开发失败)。
我们现在来创建一个程序员类的实例，好让他可以做点事情。（要记住，没有实例化的类，只是一种概念上的东西，这种东西是什么也不能做的，就象日常生活中我们所见到的一样：程序员可以开发软件，但软件是由明确的属于这个类的一个实例开发出来的，比如求伯君写的WPS，求伯君是程序员这个类的一个实例，而不是这个类本身）。下面的代码是标准的VB代码，但却是还没有实际意义的代码——因为您不要指望这段VB代码中的程序员类能为您写出一个VB程序来。

```vb
Option Explicit
'定义一个对象变量，并指定这个变量属于这个类
'WithEvents使这个对象能响映事件
Dim WithEvents MyDeveloper As cDeveloper

Private Sub Form_Load()
Set MyDeveloper = New cDeveloper '这句一定要，这是初始化这个对象
With MyDeveloper
.Name = "求伯君" '这个程序员叫做求伯君
.EmpolderTool = "C++" '这个程序员用C++进行开发
.Empolder '调用Empolder方法使这个程序员进行开发工作
End With
End Sub
'如果开发失败，则产生EmpolderLost这个事件
Private Sub MyDeveloper_EmpolderLost()
With MyDeveloper
.Study '如果开发失败,就让这个程序员进行学习
.Empolder '学习后再次进行开发
End With
End Sub
'如果开发成功，则这个程序员成为一个优秀的程序员
Private Sub MyDeveloper_EmpolderSucceed()
MsgBox "开发成功了！" & MyDeveloper.Name & _
"成为了一个优秀的程序员", vbInformation, "恭喜"
End Sub
```

我不知道该不该拿求伯君他老人家来举例，但我想，他老人家开发WPS时一定没有我写这段代码这样简单。如果这段代码您有什么不明白，没有关系，我在以后的例子中会进一步说明的，或者您也可以查查MSDN，当类，不要去查Study、.Empolder、EmpolderTool、cDeveloper这些东西，因为cDeveloper这个类和它的属性、方法和事件本来就是我想出来的，在MSDN中根本找不到对这些内容的支持。您最多能找到WithEvents这些肉容的帮助。
好了，我举了两个例子，您对类的概念应该有些认识了吧。它就是一类对象的抽象后的集合——我不知道这个定义是否是标准的，但我是这样理解的，好在这种理解并没对我在实际编写含有类模块的程序时遇到过什么麻烦。

第二天：建立一个简单的类

作者：黄河

在《第一天：类的概念》中我们讲了类的概念。今天我们就再来看看如何创建上次用过的那个cDeveloper类是怎样创建的。
首先：我们新建一个工程，然后在“工程浏览器”中按鼠标右键，在弹出菜单中选择“添加”，在添加菜单项的下级菜单中选择“添加类模块”。在弹出的“添加类模块”对话框中双击“VB 类生成器”。
接着，我们会看见类生成器对话框。现在我们点击工具栏上的第一个按钮，也就是添加新类的按钮。这时，会出现类模块生成器对话框。在这个对话框的属性选项卡上的“名称”（Name）文本框中输入cDeveloper——在这个文本框下面还有一个“基于”（Based On）文本框，这里我们先不去管它，以会我会讲到它的。在这个对话框的“特性”选项卡中，有一个“描述”文本框，我们可以在里面输入对这个类的说明，比如：“这是一个程序员类”。然后我们按确定按钮退出“类模块生成器”对话框。
再接着我们看看原来的“类生成器”对话框中左边的树型浏览器中在原来的工程下面多了一个“cDeveloper”项。选中这个项，点击对话框工具栏上的第三个按钮（添加属性），这时会弹出“属性生成器”对话框。
在这个对话框中的“属性”选项卡中的“名称”文本框中，我们输入：DeveloperName（这里有一点要注意，在“第一天”的例子中，我用的是Name属性，为什么在这里要用DeveloperName呢？因为Name是VB的保留字，如果您在这里输入了Name，VB会提示您出错的）。然后在“名称”文本框下面的“数据类型”下拉列表中选择”String”项（注意，一定要选择，如果您是输入的，因为大小写的原因，最后生成的代码可能会有问题）。下面的“声明”栏中的选项先不要管它，以后再说。
在这个对话框中的“特性”选项卡中的“描述”文本框中，您可以输入对这个属性的说明，比如：“这是程序员的姓名”。
然后按确定按钮关闭这个“属性生成器”对话框。
进行了上面的操作后，我们会发现，在“类生成器”对话框右边的属性选项卡中多了一项我们才添加的这个DeveloperName属性（您的“类生成器”对话框右边的选项卡可能没有选中“属性”这一页）。
重复上面添加属性的操作，添加一个EmpolderTool属性，数据类型还是为String。
到这时为止，我们的cDeveloper类就有了两个属性了。下面我们要为它添加两个方法.Study和Empolder。
要添加这两个方法，我们要点击“类生成器”对话框上的每四个按钮（添加新方法）。点击后会出现“方法生成器”对话框。
在这个对话框中的“属性”选项卡中的名称文本框中，我们输入“Study”（这个文本框下面的内容（包括参数）我们先不去管它）。然后点确定关闭这个“方法生成器”对话框。
重复上面添加方法的操作，添加一个Empolder方法。
最后，我们就剩添加两个事件了。让我们点击“类生成器”工具栏上的第五个按钮（添加新事件），我们就会看到“事件生成器”对话框。我们在这个对话框中的名称文本框中输入“EmpolderSucceed”，下面的参数现在不用管它，以后介绍。
重复添加新事件的操作，在我们的类中添加“EmpolderLost”事件。
现在我们的工作完成了，只要选择类生成器的“文件”菜单中的更新工程，这个cDeveloper类和它的属性、方法和事件就被添加到我们的工程中了，我们可以从“工程”浏览框中看到这个新加入的cDeveloper类模块。双击这个类模块，我们可以看见VB为这个类模块添加了下面这些代码：

```vb
Option Explicit

'local variable(s) to hold property value(s)
Private mvarDeveloperName As String 'local copy
Private mvarEmpolderTool As String 'local copy
'To fire this event, use RaiseEvent with the following syntax:
'RaiseEvent EmpolderLost[(arg1, arg2, ... , argn)]
Public Event EmpolderLost()
'To fire this event, use RaiseEvent with the following syntax:
'RaiseEvent EmpolderSucceed[(arg1, arg2, ... , argn)]
Public Event EmpolderSucceed()
Public Sub Empolder()
End Sub

Public Sub Study()
End Sub

Public Property Let EmpolderTool(ByVal vData As String)
'used when assigning a value to the property, on the left side of an assignment.
'Syntax: X.EmpolderTool = 5
mvarEmpolderTool = vData
End Property

Public Property Get EmpolderTool() As String
'used when retrieving value of a property, on the right side of an assignment.
'Syntax: Debug.Print X.EmpolderTool
EmpolderTool = mvarEmpolderTool
End Property

Public Property Let DeveloperName(ByVal vData As String)
'used when assigning a value to the property, on the left side of an assignment.
'Syntax: X.DeveloperName = 5
mvarDeveloperName = vData
End Property

Public Property Get DeveloperName() As String
'used when retrieving value of a property, on the right side of an assignment.
'Syntax: Debug.Print X.DeveloperName
DeveloperName = mvarDeveloperName
End Property
```

我使用的是英文版的VB6，所以如果您使用的是中文版的VB6，见到的代码中的注释部份和我的不同，您的是中文的注释。第一句：Option Explicit。你也可能没有，但最好请您加上，这是强制声明的选项，有了这一句之后，您的所有的变量都必须显示地声明后才能使用，这能一定程度上保证您的代码的正确性。
您如果觉得跟着我用类生成器生成这个类太繁锁，没关系，您只要象添加窗体一样新添加一个类，并双击打开它，然后把上面的这段代码Copy到它里面也可以，反正以后我会对这些生成的代码进行说明的，当您熟悉这些代码的涵义后，用代码而不用类生成器有时反而更简单。
对于这个生成的类，现在还只能对他的属性赋值，他还什么也不能做，这并不奇怪，因为我们还要添加代码告诉他该怎么做，该怎么响应事件。

下次，在学习如何添加代码来完善我们的cDeveloper的功能前，先要解决掉一些细节问题。
第三天、解决一些细节问题

在《第二天》中，我们用“类生成器”生成了一个cDeveloper类，这个类还什么也不能做，因为我们还没知诉他怎么做，这时，的cDeveloper类还象一个小婴儿，还只有一个人的样子，我们还要细心地一行一行代码地告诉他如何成为一个程序员。
在我们进一步写代码之前，我们先看看下面的一些问题——在看这些问题时，请您先把我们的类忘掉，一会儿再来说它。下面的这些问题对您也许太简单，但我认为，讲讲这些问题对理解类模块涵义的是有好处的。
问题一、子过程
您如果建立过子过程，一定知道下面语法的意义

```vb
Public Sub name [(ParameterList)]
[statements]
[Exit Sub]
[statements]
End Sub
这是创建一个子过程的语法规则，用这个规则，您可以很简单地创建一个子过程，请看下面的代码
Option Explicit
Private Sub Form_Load()
Quotient 100, 30
End Sub

Public Sub Quotient(dblDividend As Double, dblDivisor As Double)
Dim dblQuotient As Double
dblQuotient = dblDividend / dblDivisor
MsgBox "您提供的两参数相除的结果是：" & dblQuotient
End Sub
```

在上的代码中，Quotient是我们创建的一个子过程。为什么要创建这个子过程本来并不是我们在这里要讨论的问题，但我还是在这里再说几句。您也许会说把这个子过程中的内容放在Form_Load中一样可以而且还可以更简化代码。的确的，如果您的整个程序中只调用一次这种简单运算时，可以不象这样写，但如果有很多地方要用到这同一功能，而且这个功能也并不象上面代码中的简单的数学计算，而是一个相当复杂的功能时，根据程序模块化的原则，因该象上面一样把这些可重用的代码写成单独的模块，建立子过程和下面要提到的函数过程就是这其中的两种基本方法。
问题二、函数过程
我现在要对上面的代码进行一些修改：

```vb
Private Sub Form_Load()
MsgBox "您提供的两参数相除的结果是：" & Quotient(100, 30)
End Sub

Public Function Quotient(dblDividend As Double, dblDivisor As Double) As Double
Quotient = dblDividend / dblDivisor
End Function
'我们可以看到，Quotient现在已经可以返回一个值了，因为它已不再是子过程，而是一个函数过程了。
'函数过程的语法可以写成下面这样：
Public Function name [(ParameterList)] As DataType
[statements]
[Exit Function]
[statements]
End Function 
```

问题三、变量的作用范围。
当我们用Private声明一个变量时，这个变量是模块级（module level）的变量，只有这个模块中的代码可以访问这个变量，对于模块外的调用，是不可用的。
Optional. Keyword used at module level to declare constants that are available only within the module where the declaration is made. Not allowed in procedures
当我们用Public声明一个变量时，这个变量是也是模块级的变量，但他可以从所有的过程中所有的模块中调用。我们常称Public声明的变量为全局变量。
Optional. Keyword used at module level to declare constants that are available to all procedures in all modules. Not allowed in procedures.
下面，我们来重新回到我们建立的cDeveloper类模块上来。看看《第二天》中生成的那段代码吧：我把由类生成器生成的注释删除，只留下实际的代码。


```vb
Option Explicit
'下面这两句是用Private定义的cDeveloper类模块的内部变量，在这个类模块以外是无法访问这两个变量的，这两个变量是用来存储DeveloperName 和EmpolderTool的值的。
Private mvarDeveloperName As String
Private mvarEmpolderTool As String
'下面这两句是定义的两个事件，语法很简单，不是吗？如果这个事件象MouseMove这种事件一样是要带参数的， 可以用Public Event EventName(Parameter1 as ParameterType,Parameter2 as ParameterType)这种方式来声明这个事件。
Public Event EmpolderLost()
Public Event EmpolderSucceed()
'下面分别声明的是两个方法，通过上面的讨论，您可以看出来，这其实就是一个子过程。当您的方法是要返回值的时候，您就要把下而的子过程改写成一个函数过程，也就是把其中的Sub改成Function，并且声明返回值的类型，您也可以在“（）”中加入要传递进去的参数。
Public Sub Empolder()
End Sub

Public Sub Study()
End Sub
```

下面的语句可以分为两种：
一种是Property Let语句，当你对这个类的实例指定属性值时，这些Property Let语句会把您指定的值保存到这个实例的相应的内部变量（前面提到的）中。 相当于Text1.Text=strMyString
另一种是Proporty Get语句，当您读取这个类的实例的属性值时，这些Property Get语句会把您保存在它的相应的内部变量中的值读出并返回给您。相当于strMyString=Text1.Text
这两种语句对于同一属性来说一般是成对出现的。这是一种理想情况，就是说这个属性可读可写。您有没有注意到，在VB中，有些控件的属性是只读的，比如我们用Command1.name = "cmd"这一句对Command1按钮的Name属性赋值时，是不会成功的。要让我们的类也有只读属性要怎么做呢？只要删除这个属性相关的Property Let语句就可以了。

```vb
Public Property Let EmpolderTool(ByVal vData As String)
mvarEmpolderTool = vData
End Property

Public Property Get EmpolderTool() As String
EmpolderTool = mvarEmpolderTool
End Property

Public Property Let DeveloperName(ByVal vData As String)
mvarDeveloperName = vData
End Property

Public Property Get DeveloperName() As String
DeveloperName = mvarDeveloperName
End Property 
```

当然，除了Proporty Get和Property Let语句以外，您也可能会见到另一种Proporty Set语句，那是当您的类的属性的数据类型为Variant或者别的对象变量（比如ADODB.Recordset）时会用到的。
从上面的讨论，我们可以得出下面的结论：其实类模块就是由：一些变量、一些事件声明语句、一些Sub过程（为了和一般的Sub过程相区别，为了表明这是我们自定义的过程，我把它们叫做子过程）或一些函数过程、Property Let 语句、Property Get语句组成。
这次，我们了解了类生成器给我们生成的这些东西的意义了，下次，我们将着手添加代码，让这个类能真的做点事。
第四天、建立能做点事的类

下面是我们在《第二天》里建立的cDeveloper类的代码框架，我将在这段代码里直接加上一些新的代码，让它能做一点事件。请注意比较下面的代码与《第二天》中的代码有什么不同。

```vb
Option Explicit

'local variable(s) to hold property value(s)
Private mvarDeveloperName As String 'local copy
Private mvarEmpolderTool As String 'local copy
'To fire this event, use RaiseEvent with the following syntax:
'RaiseEvent EmpolderLost[(arg1, arg2, ... , argn)]
Public Event EmpolderLost()
'To fire this event, use RaiseEvent with the following syntax:
'RaiseEvent EmpolderSucceed[(arg1, arg2, ... , argn)]
Public Event EmpolderSucceed()
Public Sub Empolder()
Dim intSucceed As Integer 'Msgbox返回的是Integer变量,所以这里定义为Integer变量,您当然也可以定义它为Long,因为Long的存储空间大于Integer.
MsgBox "现在进行开发"
intSucceed = MsgBox("开发成功了吗?", vbYesNo + vbQuestion + vbDefaultButton2, "问题")
If intSucceed = vbYes Then 'vbYes是一个VB常量,它的值是一个Integer型的数据.
RaiseEvent EmpolderSucceed '如果开发成功,则激发EmpolderSucceed事件
Else
RaiseEvent EmpolderLost '如果没有开发成功,则激发EmpolderLost事件
End If
End Sub

Public Sub Study()
MsgBox mvarDeveloperName & ":请您进一步学习您的" & mvarEmpolderTool & "开发工具", vbInformation, "注意"
'在这里,您看到了对mvarDeveloperName和mvarEmpolderTool的调用,
'因为这是从类的内部调用DeveloperName属性和EmpolderTool属性,所以可以直接访问存储这两个属性的
'模块内部变量mvarDeveloperName和mvarEmpolderTool.
End Sub

Public Property Let EmpolderTool(ByVal vData As String)
'used when assigning a value to the property, on the left side of an assignment.
'Syntax: X.EmpolderTool = 5
mvarEmpolderTool = vData
End Property

Public Property Get EmpolderTool() As String
'used when retrieving value of a property, on the right side of an assignment.
'Syntax: Debug.Print X.EmpolderTool
EmpolderTool = mvarEmpolderTool
End Property

Public Property Let DeveloperName(ByVal vData As String)
'used when assigning a value to the property, on the left side of an assignment.
'Syntax: X.DeveloperName = 5
mvarDeveloperName = vData
End Property

Public Property Get DeveloperName() As String
'used when retrieving value of a property, on the right side of an assignment.
'Syntax: Debug.Print X.DeveloperName
DeveloperName = mvarDeveloperName
End Property
```

然后我们可以在工程中添加一个新窗体，这个窗体为启动窗体，在这个窗体上画一个名为Command1的按钮，用来测试我们的类。把下面的代码COPY到这个窗体的代码窗口中。

```vb
Option Explicit
Private WithEvents MyDeveloper As cDeveloper
Private Sub Command1_Click()

With MyDeveloper
.DeveloperName = "黄河"
.EmpolderTool = "C++"
.Empolder
End With

End Sub

Private Sub Form_Load()
Set MyDeveloper = New cDeveloper
End Sub

Private Sub Form_Unload(Cancel As Integer)
Set MyDeveloper = Nothing
End Sub

Private Sub MyDeveloper_EmpolderLost()
With MyDeveloper
.Study
End With
End Sub

Private Sub MyDeveloper_EmpolderSucceed()
With MyDeveloper
MsgBox .DeveloperName & ":您已经用您的" & .EmpolderTool & "工具成功完成了您的开发工作"
End With
End Sub 
```

您最好是照着这段代码自己录入到窗体中，这样您可以有更直接的理解。
比如，在这个窗体的代码窗口上方的对象下拉列表中，您可以看到MyDeveloper这一项，对它的访问方式很象一个CommandButton的实例Command1。当您在这里选中MyDeveloper时，在对象下拉列表右边的程序例表中（英文版是procedure我把它译为程序），您可以找到MyDeveloper的两个事件。这更象是一个标准的VB对象。但请注意我们在申明这个对象时，使用了WithEvents关键字，这是告诉VB，这个对象是可以接受事件的。您可以试着把WithEvents删掉，看看程序有什么变化。
然后我们运行这个程序，看看会发生什么事吧。您可以逐语句地调试这个程序，看看它是怎样运行的，这样您可以更好地理解它。
我还想说一点：您也许会说，用类编写VB程序有些么好处呀？看过了我的这几篇贴子的朋友，只要你们想一想，一定会清楚，用类编写程序，和我们以前的写程序的方式的最大的不同其实在于，我们编程的思路变了，用类编程，就是面向对象编程的最有效的方法。用以前的方式，我们的程序的每一个功能都象是由我们自己完成的（其实也是），用类编程，就好象是我们先教会一个对象所有要用到的功能，然后我们可以不去管这个对象怎样完成这些它已经学会的功能，只用考虑怎样去调用这个类。就好象一家电脑公司，以前是一切工作都是由老板一个人完成，现在新聘员工，教会他们怎样工作，然后就只用告诉这个员工要做什么工作就可以了，进一步的工作由这个员工去完成。从前面的类的例子中，我们看不出这样做的好处，但如果您的程序更大，代码更多，就如同一家大的电脑公司，培养一个甚至一批能自己完成工作的员工是很有必要的，尽管在开始培养时您会花多一点的时间，但这个过程完成后，您就会感到这样做的好处了。而且当您有了大量的自己的类库时，您以后的程序的工作量会很大程度上地减少--这该叫作人才储备吧。
还有，一个程序中可能不只一个类，而且根据您的程序的复杂程序，可能有很多的类，而且还可能有集合类。关于集合类，在以后我们再进行讨论。
因为这只是一个简单的例子，所以这个类没有什么实际的作用。在下次的贴子中，我会和您一起写一个有实际作用的类。
第五天 有实际意义的类

在我的前面的贴子中，建立了一个cDeveloper类，这个类在我们的实际编程应用中是没有什么意义的。这次，我们来建立一个有实际意义的类。
首先，我们来看一个程序片断。
程序片断的功能：读取当前Windows登录用户和计算机名。
程序片断的运行环境：Windows9x/NT/2000，局域网环境。
程序内容：1、一个窗体Form1,窗体上一个CommandButton控件Command1,代码如下：

```vb
Option Explicit
Private Declare Function GetUserName Lib "advapi32.dll" Alias "GetUserNameA" (ByVal lpBuffer As String, nSize As Long) As Long
Private Const MAX_COMPUTERNAME_LENGTH& = 15
Private Declare Function GetComputerName Lib "kernel32" Alias "GetComputerNameA" (ByVal lpBuffer As String, nSize As Long) As Long
Private Sub Command1_Click()
Dim lngReturn As Long
Dim lngSize As Long
Dim sBuffer As String
Dim lSize As Long
Dim strUserName As String
Dim strComputerName As String
'1.取得计算机名
strComputerName = String$(MAX_COMPUTERNAME_LENGTH + 1, 0)
lngSize = MAX_COMPUTERNAME_LENGTH + 1
lngReturn = GetComputerName(strComputerName, lngSize)
'2.取得Windows当前登录用户名
sBuffer = Space$(255)
lSize = Len(sBuffer)
Call GetUserName(sBuffer, lSize)
If lSize > 0 Then
strUserName = CStr(Trim(Left$(sBuffer, lSize - 1)))
Else
strUserName = vbNullString
End If
MsgBox "当前用户为：" & strUserName & " ；计算机名为：" & strComputerName
End Sub 
```

您可以看到，在这个程序中调用了两个API函数。虽然这个程序并不复杂，功能也很简单，但如果您每次要调用这个功能时，都要写这段代码，您一定还是会觉得很烦，而且API的声明调用也不是一个让一般的VB编程人员搞得很清楚的事。于是，我们可以把它封装成类，在要用到这个类的程序中，只要添加这个已经建好的类就可以了。下面是这个类的代码：（我想不出这个类的一个更好的名字，所以命名为cUser）

```vb
Option Explicit
Private Declare Function GetUserName Lib "advapi32.dll" Alias "GetUserNameA" (ByVal lpBuffer As String, nSize As Long) As Long
Private Const MAX_COMPUTERNAME_LENGTH& = 15
Private Declare Function GetComputerName Lib "kernel32" Alias "GetComputerNameA" (ByVal lpBuffer As String, nSize As Long) As Long
Private mvarComputerName As String

Public Property Get ComputerName() As String
'检索属性值时使用，位于赋值语句的右边。
'Syntax: Debug.Print X.ComputerName
Dim s$
Dim dl&
Dim sz&
If mvarComputerName = "" Then
s$ = String$(MAX_COMPUTERNAME_LENGTH + 1, 0)
sz& = MAX_COMPUTERNAME_LENGTH + 1
dl& = GetComputerName(s$, sz)
mvarComputerName = s
ComputerName = mvarComputerName
End If
End Property

'通过API，取得windows系统登录用户ID
Public Function GetTheUserName() As String
Dim sBuffer As String
Dim lSize As Long
sBuffer = Space$(255)
lSize = Len(sBuffer)
Call GetUserName(sBuffer, lSize)
If lSize > 0 Then
GetTheUserName = CStr(Trim(Left$(sBuffer, lSize - 1)))
Else
GetTheUserName = vbNullString
End If
End Function 
```

我们可以看到，这个类有一个属性ComputerName和一个方法GetTheUserName。我把它们定义为属性或方法是为了演示需要，只要您愿意，您完全可以把它们全定义为属性或全定义为方法。
还有，请您注意：Public Property Get ComputerName() As String 这一句。它没有Public Property Let ComputerName () As String 一句与之匹配。这就是我曾说过的只读属性，当一个属性只有Property Get语句时，它是只读属性，当一个属性只有Property Let语句时，它是只写属性（这种属性很少见）。
这个类没有事件，不过这没关系，以后的贴子中，我会例举有事件的类。
下面，是对这个cUser类的调用。

```vb
Private Sub Command1_Click()
Dim myUser As New cUser
MsgBox "当前用户为：" & myUser.GetTheUserName & " ；计算机名为：" & myUser.ComputerName
Set myUser = Nothing
End Sub 
```

我们可以看到，在Command1的Click事件中添加这样简单的一段代码就行了。
这个类被保存后，您可以在任何VB程序中添加并调用它，就象添加一个已经存在的窗体一样，而不用去考虑它里面的每行代码的涵义。您还可以在这个类里面封装更多的您常要用到API函数，这样，可以很大程度上简化您以后的工作。现在您是不是开始知道类的好处了？
下次，我将和您一起建立一个有数据库操作功能的类。
第六天 有数据库功能的类（一）

这次，我们来看看类在数据库应用程序中的应用。
为了测试这次的内容，您可以建一个数据库，我用的SQL Server，您也可以用ACCESS或别的数据库系统，只要ADO的连接字符串做点修改就可以了。下面是这个数据库的SQL Server建表的脚本：

```sql
CREATE TABLE [dbo].[Users] (
[ID] [int] NOT NULL ,
[UserName] [char] (10) NOT NULL ,
[UserPassword] [char] (10) NOT NULL
）
```

Go
您有没有看过没使用类的数据库应用程序？它一般是这样的，在应用程序窗体的代码中，有大量的数据库操作代码与窗体操作代码混合。比如窗体上的查询按钮Command1一般会有这样的代码：

```vb
Private Sub Command1_Click()
Dim Rs As New ADODB.Recordset
Dim strSQL As String
strSQL = "select * from Users where ID=1"
Rs.Open strSQL, conn, adOpenKeyset, adLockReadOnly, adCmdText
Text1 = Rs!UserName
Text2 = Rs!UserPassword
Text3=Rs!ID
Rs.Close
Set Rs = Nothing
End Sub
```

我并不认为这样的写法有什么不好，但如果在程序中有多个地方或者在多个程序中要对这个Users表进行查询，那么重写一部份代码是很难避免的。
下面，我们看看怎样用类来封装这种数据库功能。
首先我们创建一个类cUsers。
您会发现这个类的这段代码的比上面的复杂，因为这个类不只是有一个查询功能，它有添加记录、查询记录、修改记录、删除记录这些功能。因为时间原因，我没对这些代码进行很认真的调试和设计，所以我不保证这个类的代码能作为数据库访问的范例，只能保证您能通过它了解类在数据库访问中起到的作用，和其基本的方法。

```vb
Option Explicit
Private MyConn As ADODB.Connection , mvarUserName As String
Private mvarUserPassword As String , mvarUserID As Long
Public Event ReferSucceed(ByVal Information As String) '提交成功事件
Public Event ReferLost(ByVal Information As String) '提交失败事件
Public Function SetConn(TheConn As ADODB.Connection) As Boolean
Set MyConn = TheConn
End Function
Public Function DeleteUserInfo(Optional UserID As Long = 0) As Boolean
Dim Rs As New ADODB.Recordset
Dim strSQL As String
On Error GoTo SysErr
strSQL = "SELECT * FROM USERS WHERE UserID=" & UserID
Rs.Open strSQL, MyConn, adOpenKeyset, adLockOptimistic, adCmdText
On Error GoTo 0
If Not Rs.EOF Then
Rs.Delete
RaiseEvent ReferSucceed("删除" & UserID & "成功！")
DeleteUserInfo = True
Else
RaiseEvent ReferLost(UserID & "编号无记录！请检查您的条件")
DeleteUserInfo = False
End If
Rs.Close
Set Rs = Nothing
Exit Function
SysErr:
RaiseEvent ReferLost("意外错误！请检查您的数据库连接是否成功。")
End Function

'下面这个方法中的参数SaveType用于判断SaveUserInfo是插入新记录还是修改旧记录
'当SaveType=0时,为修改旧记录,
'当SaveType<>0时,为插入新记录.
Public Function SaveUserInfo(UserID As Long, Optional SaveType As Long = 0) As Boolean
Dim Rs As New ADODB.Recordset
Dim strSQL As String
On Error GoTo SysErr '下面这两句如果有错误,是意外错误用专门的错误处理程序处理
strSQL = "SELECT * FROM USERS WHERE UserID=" & UserID
'注意下面这一句中的LockTypeEnum参数不能用adLockReadOnly.而要用adLockOptimistic或者adLockPessimistic
Rs.Open strSQL, MyConn, adOpenKeyset, adLockOptimistic, adCmdText
On Error GoTo 0 '以上的错误是容易处理的,所以取消错误处理
If SaveType = 0 Then '如果SaveType=0则为添加新记录方式
If Rs.EOF Then
Rs.AddNew
Else
RaiseEvent ReferLost(UserID & "号用户已经存在,请重新设定您的UserID值,您的这次操作被取消!")
GoTo CloseRs
Exit Function
End If
Else '如果SaveType<>0则为修改旧记录
If Rs.EOF Then
RaiseEvent ReferLost(UserID & "号用户并不存在,请检查您的UserID值,您的这次操作被取消!")
GoTo CloseRs
Exit Function
End If
End If
Rs!UserID = mvarUserID
Rs!UserName = mvarUserName
Rs!UserPassword = mvarUserPassword
Rs.Update
If SaveType = 0 Then
RaiseEvent ReferSucceed("添加" & UserID & "成功！")
Else
RaiseEvent ReferSucceed("修改" & UserID & "成功！")
End If
SaveUserInfo = True
Exit Function
CloseRs:
Rs.Close
Set Rs = Nothing
Exit Function
SysErr:
RaiseEvent ReferLost("意外错误！请检查您的数据库连接是否成功。错误号" & Err.Number)
SaveUserInfo = False
End Function
Public Function GetUserInfo(Optional UserID As Long = 0) As Boolean
Dim Rs As New ADODB.Recordset
Dim strSQL As String
On Error GoTo SysErr
strSQL = "SELECT * FROM USERS WHERE UserID=" & UserID
Rs.Open strSQL, MyConn, adOpenKeyset, adLockReadOnly, adCmdText
On Error GoTo 0
If Not Rs.EOF Then
If Rs.RecordCount > 1 Then
RaiseEvent ReferLost("返回的记录不唯一," & UserID & "编号有" & Rs.RecordCount & "条记录！请检查您的数据库记录")
GetUserInfo = False
Else
mvarUserID = Rs!UserID
mvarUserName = Rs!UserName
mvarUserPassword = Rs!UserPassword
RaiseEvent ReferSucceed("查询" & UserID & "成功！")
GetUserInfo = True
End If
Else
RaiseEvent ReferLost(UserID & "编号无记录！请检查您的条件")
GetUserInfo = False
End If
Rs.Close
Set Rs = Nothing
Exit Function
SysErr:
RaiseEvent ReferLost("意外错误！请检查您的数据库连接是否成功。")
GetUserInfo = False
End Function
Public Property Let UserID(ByVal vData As Long)
mvarUserID = vData
End Property
Public Property Get UserID() As Long
UserID = mvarUserID
End Property
Public Property Let UserPassword(ByVal vData As String)
mvarUserPassword = vData
End Property
Public Property Get UserPassword() As String
UserPassword = mvarUserPassword
End Property
Public Property Let UserName(ByVal vData As String)
mvarUserName = vData
End Property
Public Property Get UserName() As String
UserName = mvarUserName
End Property 
```
-----------------------------------------------------------------------------
然后我们添加一个标准模块，内容如下：（请修改您的工程属性，把程序设定为从Sub Main启动。）

```vb
Option Explicit
Public conn As New ADODB.Connection
Sub main() '这是程序启动的地方
Dim strConnString As String
strConnString = "Provider=SQLOLEDB;" & _
"Persist Security Info=True;" & _
"User ID=sa;" & _
"Initial Catalog=pubs;" & _
"Data Source=ntserver"
conn.Open strConnString '打开数据库连接
Form1.Show
End Sub 
```

第六天 有数据库功能的类（二）

最后添加一个窗体：上面有三个文本框（Text1、Text2、Text3）四个CommandButton（cmdAddNew、cmdDelete、cmdQuery、cmdUpdate）好了，现在把下面的代码拷贝到这个窗体的代码部份。

```vb
Option Explicit
Public WithEvents MyUser As cUsers
Private Sub cmdAddNew_Click()
MyUser.UserID = Text1
MyUser.UserName = Text2
MyUser.UserPassword = Text3
MyUser.SaveUserInfo CLng(Text1), 0
End Sub
Private Sub cmdDelete_Click()
If MyUser.DeleteUserInfo(CLng(Text1)) Then
Text1 = ""
Text2 = ""
Text3 = ""
End If
End Sub
Private Sub cmdQuery_Click()
If MyUser.GetUserInfo(CLng(Text1)) Then
Text1 = MyUser.UserID
Text2 = MyUser.UserName
Text3 = MyUser.UserPassword
End If
End Sub
Private Sub cmdUpdate_Click()
MyUser.UserID = Text1
MyUser.UserName = Text2
MyUser.UserPassword = Text3
MyUser.SaveUserInfo CLng(Text1), 1
End Sub
Private Sub Form_Load()
Set MyUser = New cUsers
MyUser.SetConn conn '把数据库联接传递给类
End Sub
Private Sub Form_Unload(Cancel As Integer)
conn.Close
Set conn = Nothing
Set MyUser = Nothing
End Sub
Private Sub MyUser_ReferLost(ByVal Information As String)
MsgBox Information
End Sub
Private Sub MyUser_ReferSucceed(ByVal Information As String)
MsgBox Information
End Sub 
```

我问一下，您的数据库按我的格式建好了吗？如果建好了，只要把标准模块中的strConnString中的内容改为与您的实际相符的内容就可以运行这个程序了。
我们来看看上面这些代码中的一些细节问题：
一、Connection的问题：
您可能会问，这个类中封装了所有对ADO数据对象的访问，但为什么不把Connection这个对象一起放在这个类中，而要写在标准模块中呢？这个问题是这样的，当您用Connection的Open方法时，Connection对象会建立与数据库连接。这时，如果您以另一个Connection对象的Open方法，用同样的ConnectionString属性联接这个数据库时，尽管它们连接的是同一个数据库，但您的程序中也会产生两个数据库联接。因为类的实例可以不只一个，如果把Connection封装到类中，在同一个程序中的这个类的不同实例中都会产生一个与数据库的联接，这样做的结果是，造成资源的浪费。所以我们在标准模块中打开这个Connection，每个类的实例共享这一个与数据库的连接。
二、方法的返回值：
你看到了在这个cUsers类的方法中是有返回值的，而且是一个Boolean型的。我建议您的类不要用无返回值的方法，因为返回值可以使您知道这个方法的执行是否是成功的。当然您可以根据需要自己定义返回值的类型，不一定是Boolean型的。
三、用事件向用户返回信息
我们看到，在这个例子中，我们没有在类中直接用Msgbox向用户返回信息，而是把这些信息放在两个事件中，通过事件返回给用户的。这样做比用Msgbox更好的是，给类的使用者更大的灵活性，他们可以决定是否把这些信息给程序的使用者看。在事件的参数中，我们还可以加上更多的参数，为类的使用者提供更多的信息，让他们可以用更多的方法来处理这些信息。调用事件的语法是RaiseEvent EventName(ParameterList)。
四、类在数据库应用程序中使用的理由：
在数据库应用程序中使用类，可以把数据库访问和应用程序控制分离，也就是说，访问数据库方面的功能放在类里，应用程序的界面与数据库访问功能无关，这样，就可以实现，由熟悉数据库的人写类的部份，由熟悉界面的人写程序界面部份。而且这样的程序结构更合理。
我们用了这几天的时间，讨论了类的一些内容，到现在为止，如果您完全理解了这几篇贴子，应该对类有了一个比较详细地了解了。但一切在于动手，如果您只是看过这几篇贴子，而没有动手练习，那您可能有些收获，但不会对类有很深的理解的。

我们休息几天，好好理解一下这几天讨论过的内容，做些练习吧。下次，我们要讨论另一种类--集合类--的问题。下次见吧。
第七天 回味一下类的属性过程
作者：黄河[4326340]
过了有十来天了吧，因为事情比较多，所以今天才又开始我们的类的讨论。先让我们回过头来看看前几次的内容中的一些问题吧。
一、属性过程的问题。
您也许已经注意到了，在我前几次例举的类中，对类的属性使用了属性过程Property Let和Property Get，如果您会使用Public变量，一定会说：在类模块中使用一个Public变量来替代Property Let和Property Get过程不是一样的吗？比如，上次的例子中的：

```vb
Public Property Let UserName(ByVal vData As String)
mvarUserName = vData
End Property
Public Property Get UserName() As String
UserName = mvarUserName
End Property 
```

可以用：

```vb
Public UserName As String 
```

来替代，在调用类的实例时，同样可以用
MyUser.UserName = Text2 或者 Text2=MyUser.UserName 这样的语句。
您这样想是对的，对于这个UserName属性，因为它是可读可写的属性，而且对它没有别的操作，所以用一个Public变量完全可以替代它。但是我们想一想，如果这个属性是一个只读属性，或者只写属性，会怎样呢？
因为一个Public变量是可写可读的，所以用它不能解决这个问题。而用Property Let和Property Get过程，却可以解决：
只读属性（删掉Property Get语句）只用：

```vb
Public Property Let UserName(ByVal vData As String)
mvarUserName = vData
End Property 
```

只写属性（删掉Property Let语句）只用：

```vb
Public Property Let UserName(ByVal vData As String)
mvarUserName = vData
End Property 
```

我们再考虑一种情况：如果我们的类要检查用户输入的内容是否合法：比如UserName不能为空，用Public变量就不可能解决或者说不太好解决。用Property Let和Property Get过程却可以：

```vb
Public Property Let UserName(ByVal vData As String)
If vData=”” Then
Msgbox “用户名为空！”
Else
mvarUserName = vData
End If
End Property 
```

还有一种情况是只能写一次的属性。
所以，什么时候用Public变量，什么时候用属性过程，是根据您的实际情况来定的，对无限制的属性，可用Public变量，对有限制的属性用属性过程。

您可能听说过Property Set过程，而在前几次的例子中我们没有用到它。它是访问对象类型或Variant类型的属性时用的：下面的例子是MSDN中的例子，在这里借用一下。

```vb
Private mvntAnything As Variant

Public Property Get Anything() As Variant
'Set 语句只用于包含了对象引用的任何属性。
If IsObject(mvntAnything) Then
Set Anything = mvntAnything
Else
Anything = mvntAnything
End If
End Property

Public Property Let Anything(ByVal NewValue As Variant)
'（省略验证代码。）
mvntAnything = NewWidget
End Property

Public Property Set Anything(ByVal NewValue As Variant)
'（省略验证代码。）
Set mvntAnything = NewWidget
End Property 
```

如果您学过C++，那我说上面的代码中的Property Set 和Property Let相当于函数的重载，您一定不会奇怪，所谓函数的重载，按我的理解就是：两个函数过程的名称相同，但这个函数要处理的数据类型不同，在调用这个函数的名称时，系统知道该访问哪一个函数过程。这个Property Set 和Property Let就是这样的，同样是Anything这一句称，同样是对类的属性设定值，但当传入的数据是对象类型时，系统会调用Property Set过程，当传入的数据不是一个对象，而是一个Long或String等类型时，系统会调用Property Let语句。这时，Property Get也有一点不同，多了一个IsObject判断。当您使用VB6的类生成器生成类的属性时，这三种属性过程是由VB6给您生成的，但当您是手工建立类代码时，一定要注意您的属性的数据类型。

二、Public WithEvents MyUser As cUsers 中的一点问题
这里的MyUser，我们称之为WithEvents 变量，我们要注意VB对 WithEvents 变量的一些限制：
WithEvents 变量不能是派生对象变量。也就是说，不能把它声明为 As Object ，比如：Public WithEvents MyUser As Object ，当声明该变量时必须指定类名。
不能把 WithEvents 变量声明为 As New。比如：Public WithEvents MyUser As New cUsers ，必须明确地用Set MyUser=New cUser创建事件源对象，并将它赋给 WithEvents 变量。
不能在标准模块中声明 WithEvents 变量。只能在类模块、窗体模块以及其它定义类的模块中声明。
不能创建 WithEvents 变量数组。

在看这些内容时，请对照MSDN。要注意：中文MSDN在翻译时，有一些错误和漏掉的东西，这些错误虽然不是关键的，但对我们理解它带来一些问题，如果您的英文水平不太好，可以找一套三张碟的英文MSDN对照一下——这样也可以使您在学VB时复习一下英文——我现在就是这样干的。
下次，我们一起讨论集合类。
第八天 为创建集合类做些准备

什么是集合？比如我们在前面创建的cUsers类，当我们用
Public WithEvents MyUser As cUsers
Set MyUser=New cUser
就创建了这个类的一个实例——这个实例就是一个对象。但是我们的User往往不只一个，可能有叫黄河的用户，可能有叫快乐老猫的用户，于是我们就可以用这样的方法创建多个User

```vb
Public WithEvents MyUser1 As cUsers
Public WithEvents MyUser2 As cUsers
…………
Set MyUser1=New cUser
Set MyUser2=New cUser
…………
```

我们会发现，这些User有着非常相似的地方——我不是说他们长得相似，而是说它们都有.UserID 、 MyUser.UserName 、MyUser.UserPassword 这些属性，也有相似的方法和事件。但是用MyUser1、MyUser2这样的方式来管理这些User并不方便。
您也许会想到可以用一个对象数组MyUser()来管理他们，但是在第七天里的“Public WithEvents MyUser As cUsers 中的一点问题”中我们曾谈到：不能创建 WithEvents 变量数组。也就是说，如果用数组来管理这些User，那么，我们就会失去对象事件的处理方法——对于面向对象编程来说，这是一个重大的损失。
您还会想到的一定就是用Collection（集合）对象来管理这些User了。集合就是把一些对象按一定的方式组织在一起。的我们看看下面的代码：还是使用上面用到过的cUser类，在窗体部份加上下面的代码

```vb
Option Explicit
Public WithEvents MyUser As cUsers
Public colUser As Collection
Private Sub cmdAddNew_Click()
Set MyUser = New cUsers '创建一个用户
MyUser.UserID = Text1
MyUser.UserName = Text2
MyUser.UserPassword = Text3
colUser.Add MyUser, Text1
Set MyUser = Nothing
End Sub

Private Sub cmdDelete_Click()
colUser.Remove CLng(Text1)
Text1 = ""
Text2 = ""
Text3 = ""
End Sub

Private Sub cmdQuery_Click()
Set MyUser = New cUsers
On Error GoTo Err
Set MyUser = colUser(Text1)
On Error GoTo 0
Text1 = MyUser.UserID
Text2 = MyUser.UserName
Text3 = MyUser.UserPassword
Set MyUser = Nothing
Exit Sub
Err:
MsgBox "没有这个用户"
End Sub

Private Sub cmdUpdate_Click()
Set MyUser = New cUsers
On Error GoTo Err
Set MyUser = colUser(Text1)
On Error GoTo 0
MyUser.UserID = Text1
MyUser.UserName = Text2
MyUser.UserPassword = Text3
Set MyUser = Nothing
Exit Sub
Err:
MsgBox "无此用户"
End Sub

Private Sub Form_Load()
Set colUser = New Collection '在这里创建这个集合的一个实例
End Sub

Private Sub Form_Unload(Cancel As Integer)
Set MyUser = Nothing
Set colUser = Nothing
End Sub 
```

在这里，集合colUser在程序运行期间起到了数据库的作用，它保存了所有User对象。您可能会发现，在这里，我没用到cUser类的GetUserInfo、SaveUserInfo、DeleteUserInfo方法，因为这些操作由集合来完成了，如果要保存这些用户的情况，可以在窗体中添加对数据库的操作。
但是这样做对吗？这样做符合对象封装的要求吗？其实这样做的确是相当不正确的。我们在前面讨论过的类的优点在这里都没发挥出来，在这里，这个类更象是一个用户类型：

```vb
Private Type MyUser
    UserID As Long
    UserName As String
    Password As String
End Type 
```

而且还有一个问题，就是这个集合不能判断存到它里面去的对象是什么，比如前面的colUser.Add MyUser, Text1这一句，我们把它改成colUser.Add Me, Text1，就是把当前窗体存入这个集合。这时，不会报错，但当您用For Each……Next或者用colUser(Index)来访问这个集合时，当正好访问到这个Me时，Me.UserID = Text1一定会出错的。
那么有什么更好的办法呢？那就是集合类——就是把集合封装到一个类中，用这个集合类来处理它的子类。
比如，我们的cUsers类，我们把它更名为cUser，用来处理单个的User对象，再建一个集合类，名为cUsers，用来存储所有被创建的User对象的集合。
今天就到这里，明天我们就写一个完整的包涵集合类的程序。


在看这部份内容时，您可以参考MSDN中的程序员指南中的VB能做什么中的用对象编程部份。从技术水平上讲，MSDN的编写者才是高手，只不过他们在语言表达上，也就理所当然地显得比我高深，理解他们的文章就有了一些难度。

