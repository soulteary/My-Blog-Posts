# [VB]转载 - 模拟键盘事件总结

用VB模拟键盘事件的N种方法

CSDN上一贴强文，看完之后不禁七窍多通一窍:)

<!-- more -->

 键盘是我们使用计算机的一个很重要的输入设备了，即使在鼠标大行其道的今天，很多程序依然离不开键盘来操作。但是有时候，一些重复性的，很繁琐的键盘操作总会让人疲惫，于是就有了用程序来代替人们按键的方法，这样可以把很多重复性的键盘操作交给程序来模拟，省了很多精力，按键精灵就是这样的一个软件。那么我们怎样才能用VB来写一个程序，达到与按键精灵类似的功能呢？那就让我们来先了解一下windows中响应键盘事件的机制。
 当用户按下键盘上的一个键时，键盘内的芯片会检测到这个动作，并把这个信号传送到计算机。如何区别是哪一个键被按下了呢？键盘上的所有按键都有一个编码，称作键盘扫描码。当你按下一个键时，这个键的扫描码就被传给系统。扫描码是跟具体的硬件相关的，同一个键，在不同键盘上的扫描码有可能不同。键盘控制器就是将这个扫描码传给计算机，然后交给键盘驱动程序。键盘驱动程序会完成相关的工作，并把这个扫描码转换为键盘虚拟码。什么是虚拟码呢？因为扫描码与硬件相关，不具有通用性，为了统一键盘上所有键的编码，于是就提出了虚拟码概念。无论什么键盘，同一个按键的虚拟码总是相同的，这样程序就可以识别了。简单点说，虚拟码就是我们经常可以看到的像VK_A,VK_B这样的常数，比如键A的虚拟码是65，写成16进制就是&H41，注意，人们经常用16进制来表示虚拟码。当键盘驱动程序把扫描码转换为虚拟码后，会把这个键盘操作的扫描码和虚拟码还有其它信息一起传递给操作系统。然后操作系统则会把这些信息封装在一个消息中，并把这个键盘消息插入到消息列队。最后，要是不出意外的话，这个键盘消息最终会被送到当前的活动窗口那里，活动窗口所在的应用程序接收到这个消息后，就知道键盘上哪个键被按下，也就可以决定该作出什么响应给用户了。这个过程可以简单的如下表示：
 用户按下按键-----键盘驱动程序将此事件传递给操作系统-----操作系统将键盘事件插入消息队列-----键盘消息被发送到当前活动窗口
 明白了这个过程，我们就可以编程实现在其中的某个环节来模拟键盘操作了。在VB中，有多种方法可以实现键盘模拟，我们就介绍几种比较典型的。

 1.局部级模拟

 从上面的流程可以看出，键盘事件是最终被送到活动窗口，然后才引起目标程序响应的。那么最直接的模拟方法就是：直接伪造一个键盘消息发给目标程序。哈哈，这实在是很简单，windows提供了几个这样的API函数可以实现直接向目标程序发送消息的功能，常用的有SendMessage和PostMessage，它们的区别是PostMessage函数直接把消息仍给目标程序就不管了，而SendMessage把消息发出去后，还要等待目标程序返回些什么东西才好。这里要注意的是，模拟键盘消息一定要用PostMessage函数才好，用SendMessage是不正确的(因为模拟键盘消息是不需要返回值的，不然目标程序会没反应)，切记切记！PostMessage函数的VB声明如下：

```vb
Declare Function PostMessage Lib "user32" Alias "PostMessageA" (ByVal hwnd As Long, ByVal wMsg As Long, ByVal wParam As Long, lParam As Any) As Long
```

参数
hwnd 是你要发送消息的目标程序上某个控件的句柄，参数wMsg 是消息的类型，表示你要发送什么样的消息，最后wParam 和lParam 这两个参数是随消息附加的数据，具体内容要由消息决定。
再来看看wMsg 这个参数，要模拟按键就靠这个了。键盘消息常用的有如下几个：
WM_KEYDOWN 表示一个普通键被按下
WM_KEYUP 表示一个普通键被释放
WM_SYSKEYDOWN 表示一个系统键被按下，比如Alt键
WM_SYSKEYUP 表示一个系统键被释放，比如Alt键
如果你确定要发送以上几个键盘消息，那么再来看看如何确定键盘消息中的wParam 和lParam 这两个参数。在一个键盘消息中，wParam 参数的含义较简单，它表示你要发送的键盘事件的按键虚拟码，比如你要对目标程序模拟按下A键，那么wParam 参数的值就设为VK_A ,至于lParam 这个参数就比较复杂了，因为它包含了多个信息，一般可以把它设为0，但是如果你想要你的模拟更真实一些，那么建议你还是设置一下这个参数。那么我们就详细了解一下lParam 吧。lParam 是一个long类型的参数，它在内存中占4个字节，写成二进制就是00000000 00000000 00000000 00000000 一共是32位，我们从右向左数，假设最右边那位为第0位(注意是从0而不是从1开始计数)，最左边的就是第31位，那么该参数的的0-15位表示键的发送次数等扩展信息，16-23位为按键的扫描码，24-31位表示是按下键还是释放键。大家一般习惯写成16进制的，那么就应该是&H00 00 00 00 ，第0-15位一般为&H0001，如果是按下键，那么24-31位为&H00，释放键则为&HC0,那么16-23位的扫描码怎么会得呢？这需要用到一个API函数MapVirtualKey，这个函数可以将虚拟码转换为扫描码，或将扫描码转换为虚拟码，还可以把虚拟码转换为对应字符的ASCII码。它的VB声明如下：

```vb
Declare Function MapVirtualKey Lib "user32" Alias "MapVirtualKeyA" (ByVal wCode As Long, ByVal wMapType As Long) As Long
```

参数
wCode 表示待转换的码，参数wMapType 表示从什么转换为什么，如果是虚拟码转扫描码，则wMapType 设置为0，如果是虚拟扫描码转虚拟码，则wMapType 设置为1，如果是虚拟码转ASCII码，则wMapType 设置为2.相信有了这些，我们就可以构造键盘事件的lParam参数了。下面给出一个构造lParam参数的函数：

```vb
Declare Function MapVirtualKey Lib "user32" Alias "MapVirtualKeyA" (ByVal wCode As Long, ByVal wMapType As Long) As Long
Function MakeKeyLparam(ByVal VirtualKey As Long, ByVal flag As Long) As Long
'参数
VirtualKey表示按键虚拟码,flag表示是按下键还是释放键，用WM_KEYDOWN和WM_KEYUP这两个常数表示
Dim s As String
Dim Firstbyte As String 'lparam参数的24-31位
If flag = WM_KEYDOWN Then '如果是按下键
Firstbyte = "00"
Else
Firstbyte = "C0" '如果是释放键
End If
Dim Scancode As Long
'获得键的扫描码
Scancode = MapVirtualKey(VirtualKey, 0)
Dim Secondbyte As String 'lparam参数的16-23位，即虚拟键扫描码
Secondbyte = Right("00" & Hex(Scancode), 2)
s = Firstbyte & Secondbyte & "0001" '0001为lparam参数的0-15位，即发送次数和其它扩展信息
MakeKeyLparam = Val("&H" & s)
End Function
```

这个函数像这样调用，比如按下A键，那么lParam=MakeKeyLparam(VK_A,WM_KEYDOWN) ，很简单吧。值得注意的是，即使你发送消息时设置了lParam参数的值，但是系统在传递消息时仍然可能会根据当时的情况重新设置该参数，那么目标程序收到的消息中lParam的值可能会和你发送时的有所不同。所以，如果你很懒的话，还是直接把它设为0吧，对大多数程序不会有影响的，呵呵。
好了，做完以上的事情，现在我们可以向目标程序发送键盘消息了。首先取得目标程序接受这个消息的控件的句柄，比如目标句柄是12345，那么我们来对目标模拟按下并释放A键，像这样：(为了简单起见，lParam这个参数就不构造了，直接传0)

```vb
PostMessage 12345，WM_KEYDOWN，VK_A，0& '按下A键
PostMessage 12345，WM_UP，VK_A，0& '释放A键
```

好了，一次按键就完成了。现在你可以迫不及待的打开记事本做实验，先用FindWindowEx这类API函数找到记事本程序的句柄，再向它发送键盘消息，期望记事本里能诡异的自动出现字符。可是你马上就是失望了，咦，怎么一点反应也没有？你欺骗感情啊~~~~~~~~~~55555555555555 不是的哦，接着往下看啊。
一般目标程序都会含有多个控件，并不是每个控件都会对键盘消息作出反应，只有把键盘消息发送给接受它的控件才会得到期望的反应。那记事本来说，它的编辑框其实是个edit类，只有这个控件才对键盘事件有反应，如果只是把消息发给记事本的窗体，那是没有用的。现在你找出记事本那个编辑框的句柄，比如是54321，那么写如下代码：
PostMessage 54321，WM_KEYDOWN，VK_F1，0& '按下F1键
PostMessage 54321，WM_UP，VK_F1，0& '释放F1键
怎么样，是不是打开了记事本的“帮助”信息？这说明目标程序已经收到了你发的消息，还不错吧~~~~~~~~
可以马上新问题就来了，你想模拟向记事本按下A这个键，好在记事本里自动输入字符，可是，没有任何反应！这是怎么一回事呢？
原来，如果要向目标程序发送字符，光靠WM_KEYDOWN和WM_UP这两个事件还不行，还需要一个事件：WM_CHAR，这个消息表示一个字符，程序需靠它看来接受输入的字符。一般只有A，B，C等这样的按键才有WM_CHAR消息，别的键(比如方向键和功能键)是没有这个消息的，WM_CHAR消息一般发生在WM_KEYDOWN消息之后。WM_CHAR消息的lParam参数的含义与其它键盘消息一样，而它的wParam则表示相应字符的ASCII编码(可以输入中文的哦^_^)，现在你可以写出一个完整的向记事本里自动写入字符的程序了，下面是一个例子，并附有这些消息常数的具体值：

```vb
Declare Function PostMessage Lib "user32" Alias "PostMessageA" (ByVal hwnd As Long, ByVal wMsg As Long, ByVal wParam As Long, lParam As Any) As Long
Declare Function MapVirtualKey Lib "user32" Alias "MapVirtualKeyA" (ByVal wCode As Long, ByVal wMapType As Long) As Long

Public Const WM_KEYDOWN = &H100
Public Const WM_KEYUP = &H101
Public Const WM_CHAR = &H102
Public Const VK_A = &H41

Function MakeKeyLparam(ByVal VirtualKey As Long, ByVal flag As Long) As Long
Dim s As String
Dim Firstbyte As String 'lparam参数的24-31位
If flag = WM_KEYDOWN Then '如果是按下键
Firstbyte = "00"
Else
Firstbyte = "C0" '如果是释放键
End If
Dim Scancode As Long
'获得键的扫描码
Scancode = MapVirtualKey(VirtualKey, 0)
Dim Secondbyte As String 'lparam参数的16-23位，即虚拟键扫描码
Secondbyte = Right("00" & Hex(Scancode), 2)
s = Firstbyte & Secondbyte & "0001" '0001为lparam参数的0-15位，即发送次数和其它扩展信息
MakeKeyLparam = Val("&H" & s)
End Function

Private Sub Form_Load()
dim hwnd as long
hwnd = XXXXXX 'XXXXX表示记事本编辑框的句柄
PostMessage hwnd,WM_KEYDOWN，VK_A，MakeKeyLparam(VK_A,WM_KEYDOWN) '按下A键
PostMessage hwnd,WM_CHAR，ASC("A"),MakeKeyLparam(VK_A,WM_KEYDOWN) '输入字符A
PostMessage hwnd,WM_UP，VK_A，MakeKeyLparam(VK_A,WM_UP) '释放A键
End Sub
```

这就是通过局部键盘消息来模拟按键。这个方法有一个极大的好处，就是：它可以实现后台按键，也就是说他对你的前台操作不会有什么影响。比如，你可以用这个方法做个程序在游戏中模拟按键来不断地执行某些重复的操作，而你则一边喝茶一边与QQ上的MM们聊得火热，它丝毫不会影响你的前台操作。无论目标程序是否获得焦点都没有影响，这就是后台模拟按键的原理啦~~~~
2.全局级模拟

你会发现，用上面的方法模拟按键并不是对所有程序都有效的，有的程序啊，你向它发了一大堆消息，可是它却一点反应也没有。这是怎么回事呢？这就要看具体的情况了，有些程序(特别是一些游戏)出于某些原因，会禁止用户对它使用模拟按键程序，这个怎么实现呢？比如可以在程序中检查一下，如果发现自己不是活动窗口，就不接受键盘消息。或者仔细检查一下收到的键盘消息，你会发现真实的按键和模拟的按键消息总是有一些小差别，从这些小差别上，目标程序就能判断出：这是假的！是伪造的！！因此，如果用PostMessage发送局部消息模拟按键不成功的话，你可以试一试全局级的键盘消息，看看能不能骗过目标程序。
模拟全局键盘消息常见的可以有以下一些方法：
(1) 用API函数keybd_event，这个函数可以用来模拟一个键盘事件，它的VB声明为：

```vb
Declare Sub keybd_event Lib "user32" (ByVal bVk As Byte, ByVal bScan As Byte, ByVal dwFlags As Long, ByVal dwExtraInfo As Long)
```

参数
bVk表示要模拟的按键的虚拟码，bScan表示该按键的扫描码(一般可以传0)，dwFlags表示是按下键还是释放键(按下键为0，释放键为2)，dwExtraInfo是扩展标志，一般没有用。比如要模拟按下A键，可以这样：

```vb
Const KEYEVENTF_KEYUP = &H2
keybd_event VK_A, 0, 0, 0 '按下A键
keybd_event VK_A, 0, KEYEVENTF_KEYUP, 0 '释放A键
```

注意有时候按键的速度不要太快，否则会出问题，可以用API函数Sleep来进行延时，声明如下：

```vb
Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long)
```

参数dwMilliseconds表示延时的时间，以毫秒为单位。
那么如果要模拟按下功能键怎么做呢？比如要按下Ctrl+C实现拷贝这个功能，可以这样：

```vb
keybd_event VK_Ctrl, 0, 0, 0 '按下Ctrl键
keybd_event VK_C, 0, 0, 0 '按下C键
Sleep 500 '延时500毫秒
keybd_event VK_C, 0, KEYEVENTF_KEYUP, 0 '释放C键
keybd_event VK_Ctrl, 0, KEYEVENTF_KEYUP, 0 '释放Ctrl键
```

好了，现在你可以试试是不是可以骗过目标程序了，这个函数对大部分的窗口程序都有效，可是仍然有一部分游戏对它产生的键盘事件熟视无睹，这时候，你就要用上bScan这个参数了。一般的，bScan都传0，但是如果目标程序是一些DirectX游戏，那么你就需要正确使用这个参数传入扫描码，用了它可以产生正确的硬件事件消息，以被游戏识别。这样的话，就可以写成这样：

```vb
keybd_event VK_A, MapVirtualKey(VK_A, 0), 0, 0 '按下A键
keybd_event VK_A, MapVirtualKey(VK_A, 0), KEYEVENTF_KEYUP, 0 '释放A键
```

以上就是用keybd_event函数来模拟键盘事件。除了这个函数，SendInput函数也可以模拟全局键盘事件。SendInput可以直接把一条消息插入到消息队列中，算是比较底层的了。它的VB声明如下：

```vb
Declare Function SendInput Lib "user32.dll" (ByVal nInputs As Long, pInputs As GENERALINPUT, ByVal cbSize As Long) As Long
```

参数：
nlnprts：定义plnputs指向的结构的数目。
plnputs：指向INPUT结构数组的指针。每个结构代表插人到键盘或鼠标输入流中的一个事件。
cbSize：定义INPUT结构的大小。若cbSize不是INPUT结构的大小，则函数调用失败。
返回值：函数返回被成功地插人键盘或鼠标输入流中的事件的数目。若要获得更多的错误信息，可以调用GetlastError函数。
备注：Sendlnput函数将INPUT结构中的事件顺序地插入键盘或鼠标的输入流中。这些事件与用户插入的（用鼠标或键盘）或调用keybd_event，mouse_event，或另外的Sendlnput插人的键盘或鼠标的输入流不兼容。
嗯，这个函数用起来蛮复杂的，因为它的参数都是指针一类的东西。要用它来模拟键盘输入，先要构造一组数据结构，把你要模拟的键盘消息装进去，然后传给它。为了方便起见，把它做在一个过程里面，要用的时候直接调用好了，代码如下：

```vb
Declare Function SendInput Lib "user32.dll" (ByVal nInputs As Long, pInputs As GENERALINPUT, ByVal cbSize As Long) As Long
Declare Sub CopyMemory Lib "kernel32" Alias "RtlMoveMemory" (pDst As Any, pSrc As Any, ByVal ByteLen As Long)
Type GENERALINPUT
dwType As Long
xi(0 To 23) As Byte
End Type

Type KEYBDINPUT
wVk As Integer
wScan As Integer
dwFlags As Long
time As Long
dwExtraInfo As Long
End Type

Const INPUT_KEYBOARD = 1

Sub MySendKey(bkey As Long)
'参数bkey传入要模拟按键的虚拟码即可模拟按下指定键
Dim GInput(0 To 1) As GENERALINPUT
Dim KInput As KEYBDINPUT
KInput.wVk = bkey '你要模拟的按键
KInput.dwFlags = 0 '按下键标志
GInput(0).dwType = INPUT_KEYBOARD
CopyMemory GInput(0).xi(0), KInput, Len(KInput) '这个函数用来把内存中KInput的数据复制到GInput
KInput.wVk = bkey
KInput.dwFlags = KEYEVENTF_KEYUP ' 释放按键
GInput(1).dwType = INPUT_KEYBOARD ' 表示该消息为键盘消息
CopyMemory GInput(1).xi(0), KInput, Len(KInput)
'以上工作把按下键和释放键共2条键盘消息加入到GInput数据结构中
SendInput 2, GInput(0), Len(GInput(0)) '把GInput中存放的消息插入到消息列队
End Sub
```

除了以上这些，用全局钩子也可以模拟键盘消息。如果你对windows中消息钩子的用法已经有所了解，那么你可以通过设置一个全局HOOK来模拟键盘消息，比如，你可以用WH_JOURNALPLAYBACK这个钩子来模拟按键。WH_JOURNALPLAYBACK是一个系统级的全局钩子，它和WH_JOURNALRECORD的功能是相对的，常用它们来记录并回放键盘鼠标操作。WH_JOURNALRECORD钩子用来将键盘鼠标的操作忠实地记录下来，记录下来的信息可以保存到文件中，而WH_JOURNALPLAYBACK则可以重现这些操作。当然亦可以单独使用WH_JOURNALPLAYBACK来模拟键盘操作。你需要首先声明SetWindowsHookEx函数，它可以用来安装消息钩子：

```vb
Declare Function SetWindowsHookEx Lib "user32" Alias "SetWindowsHookExA" (ByVal idHook As Long,ByVal lpfn As Long, ByVal hmod As Long, ByVal dwThreadId As Long) As Long
```

先安装WH_JOURNALPLAYBACK这个钩子，然后你需要自己写一个钩子函数，在系统调用它时，把你要模拟的事件传递给钩子参数lParam所指向的EVENTMSG区域，就可以达到模拟按键的效果。不过用这个钩子模拟键盘事件有一个副作用，就是它会锁定真实的鼠标键盘，不过如果你就是想在模拟的时候不会受真实键盘操作的干扰，那么用用它倒是个不错的主意。
3.驱动级模拟

如果上面的方法你都试过了，可是你发现目标程序却仍然顽固的不接受你模拟的消息，寒~~~~~~~~~还好，我还剩下最后一招，这就是驱动级模拟：直接读写键盘的硬件端口！
有一些使用DirectX接口的游戏程序，它们在读取键盘操作时绕过了windows的消息机制，而使用DirectInput.这是因为有些游戏对实时性控制的要求比较高，比如赛车游戏，要求以最快速度响应键盘输入。而windows消息由于是队列形式的，消息在传递时会有不少延迟，有时1秒钟也就传递十几条消息，这个速度达不到游戏的要求。而DirectInput则绕过了windows消息，直接与键盘驱动程序打交道，效率当然提高了不少。因此也就造成，对这样的程序无论用PostMessage或者是keybd_event都不会有反应，因为这些函数都在较高层。对于这样的程序，只好用直接读写键盘端口的方法来模拟硬件事件了。要用这个方法来模拟键盘，需要先了解一下键盘编程的相关知识。
在DOS时代，当用户按下或者放开一个键时，就会产生一个键盘中断(如果键盘中断是允许的)，这样程序会跳转到BIOS中的键盘中断处理程序去执行。打开windows的设备管理器，可以查看到键盘控制器由两个端口控制。其中&H60是数据端口，可以读出键盘数据，而&H64是控制端口，用来发出控制信号。也就是，从&H60号端口可以读此键盘的按键信息，当从这个端口读取一个字节，该字节的低7位就是按键的扫描码，而高1位则表示是按下键还是释放键。当按下键时，最高位为0，称为通码，当释放键时，最高位为1，称为断码。既然从这个端口读数据可以获得按键信息，那么向这个端口写入数据就可以模拟按键了！用过QbASIC4.5的朋友可能知道，QB中有个OUT命令可以向指定端口写入数据，而INP函数可以读取指定端口的数据。那我们先看看如果用QB该怎么写代码：
假如你想模拟按下一个键，这个键的扫描码为&H50，那就这样

```vb
OUT &H64,&HD2 '把数据&HD2发送到&H64端口。这是一个KBC指令，表示将要向键盘写入数据
OUT &H60,&H50 '把扫描码&H50发送到&H60端口，表示模拟按下扫描码为&H50的这个键
```

那么要释放这个键呢？像这样，发送该键的断码：

```vb
OUT &H64,&HD2 '把数据&HD2发送到&H64端口。这是一个KBC指令，表示将要向键盘写入数据
OUT &H60,(&H50 OR &H80) '把扫描码&H50与数据&H80进行或运算，可以把它的高位置1，得到断码，表示释放这个键
```

好了，现在的问题就是在VB中如何向端口写入数据了。因为在windows中，普通应用程序是无权操作端口的，于是我们就需要一个驱动程序来帮助我们实现。在这里我们可以使用一个组件WINIO来完成读写端口操作。什么是WINIO？WINIO是一个全免费的、无需注册的、含源程序的WINDOWS2000端口操作驱动程序组件(可以到http://www.internals.com/上去下载)。它不仅可以操作端口，还可以操作内存；不仅能在VB下用，还可以在DELPHI、VC等其它环境下使用，性能特别优异。下载该组件，解压缩后可以看到几个文件夹，其中Release文件夹下的3个文件就是我们需要的，这3个文件是WinIo.sys(用于win xp下的驱动程序)，WINIO.VXD(用于win 98下的驱动程序)，WinIo.dll(封装函数的动态链接库)，我们只需要调用WinIo.dll中的函数，然后WinIo.dll就会安装并调用驱动程序来完成相应的功能。值得一提的是这个组件完全是绿色的，无需安装，你只需要把这3个文件复制到与你的程序相同的文件夹下就可以使用了。用法很简单，先用里面的InitializeWinIo函数安装驱动程序，然后就可以用GetPortVal来读取端口或者用SetPortVal来写入端口了。好，让我们来做一个驱动级的键盘模拟吧。先把winio的3个文件拷贝到你的程序的文件夹下，然后在VB中新建一个工程，添加一个模块，在模块中加入下面的winio函数声明:

```vb
Declare Function MapPhysToLin Lib "WinIo.dll" (ByVal PhysAddr As Long, ByVal PhysSize As Long, ByRef PhysMemHandle) As Long
Declare Function UnmapPhysicalMemory Lib "WinIo.dll" (ByVal PhysMemHandle, ByVal LinAddr) As Boolean
Declare Function GetPhysLong Lib "WinIo.dll" (ByVal PhysAddr As Long, ByRef PhysVal As Long) As Boolean
Declare Function SetPhysLong Lib "WinIo.dll" (ByVal PhysAddr As Long, ByVal PhysVal As Long) As Boolean
Declare Function GetPortVal Lib "WinIo.dll" (ByVal PortAddr As Integer, ByRef PortVal As Long, ByVal bSize As Byte) As Boolean
Declare Function SetPortVal Lib "WinIo.dll" (ByVal PortAddr As Integer, ByVal PortVal As Long, ByVal bSize As Byte) As Boolean
Declare Function InitializeWinIo Lib "WinIo.dll" () As Boolean
Declare Function ShutdownWinIo Lib "WinIo.dll" () As Boolean
Declare Function InstallWinIoDriver Lib "WinIo.dll" (ByVal DriverPath As String, ByVal Mode As Integer) As Boolean
Declare Function RemoveWinIoDriver Lib "WinIo.dll" () As Boolean

' ------------------------------------以上是WINIO函数声明-------------------------------------------

Declare Function MapVirtualKey Lib "user32" Alias "MapVirtualKeyA" (ByVal wCode As Long, ByVal wMapType As Long) As Long

'-----------------------------------以上是WIN32 API函数声明-----------------------------------------

再添加下面这个过程：
Sub KBCWait4IBE() '等待键盘缓冲区为空
Dim dwVal As Long
Do
GetPortVal &H64, dwVal, 1
'这句表示从&H64端口读取一个字节并把读出的数据放到变量dwVal中
'GetPortVal函数的用法是GetPortVal 端口号,存放读出数据的变量,读入的长度
Loop While (dwVal And &H2)
End Sub
上面的是一个根据KBC规范写的过程，它的作用是在向键盘端口写入数据前等待一段时间，后面将会用到。
然后再添加如下过程，这2个过程用来模拟按键：

Public Const KBC_KEY_CMD = &H64 '键盘命令端口
Public Const KBC_KEY_DATA = &H60 '键盘数据端口

Sub MyKeyDown(ByVal vKeyCoad As Long)
'这个用来模拟按下键，参数vKeyCoad传入按键的虚拟码
Dim btScancode As Long
btScancode = MapVirtualKey(vKeyCoad, 0)

KBCWait4IBE '发送数据前应该先等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
'SetPortVal函数用于向端口写入数据，它的用法是SetPortVal 端口号,欲写入的数据，写入数据的长度
KBCWait4IBE
SetPortVal KBC_KEY_DATA, btScancode, 1 '写入按键信息,按下键

End Sub

Sub MyKeyUp(ByVal vKeyCoad As Long)
'这个用来模拟释放键，参数vKeyCoad传入按键的虚拟码
Dim btScancode As Long
btScancode = MapVirtualKey(vKeyCoad, 0)

KBCWait4IBE '等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
KBCWait4IBE
SetPortVal KBC_KEY_DATA, (btScancode Or &H80), 1 '写入按键信息，释放键

End Sub
定义了上面的过程后，就可以用它来模拟键盘输入了。在窗体模块中添加一个定时器控件，然后加入以下代码：

Private Sub Form_Load()

If InitializeWinIo = False Then
'用InitializeWinIo函数加载驱动程序，如果成功会返回true，否则返回false
MsgBox "驱动程序加载失败!"
Unload Me
End If
Timer1.Interval=3000
Timer1.Enabled=True
End Sub

Private Sub Form_Unload(Cancel As Integer)
ShutdownWinIo '程序结束时记得用ShutdownWinIo函数卸载驱动程序
End Sub

Private Sub Timer1_Timer()
Dim VK_A as Long = &H41
MyKeyDown VK_A
MyKeyUp VK_A '模拟按下并释放A键
End Sub
```

运行上面的程序，就会每隔3秒钟模拟按下一次A键，试试看，怎么样，是不是对所有程序都有效果了？
需要注意的问题：
要在VB的调试模式下使用WINIO，需要把那3个文件拷贝到VB的安装目录中。
键盘上有些键属于扩展键(比如键盘上的方向键就是扩展键)，对于扩展键不应该用上面的MyKeyDown和MyKeyUp过程来模拟，可以使用下面的2个过程来准确模拟扩展键：

```vb
Sub MyKeyDownEx(ByVal vKeyCoad As Long) '模拟扩展键按下，参数vKeyCoad是扩展键的虚拟码
Dim btScancode As Long
btScancode = MapVirtualKey(vKeyCoad, 0)

KBCWait4IBE '等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
KBCWait4IBE
SetPortVal KBC_KEY_DATA, &HE0, 1 '写入扩展键标志信息
KBCWait4IBE '等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
KBCWait4IBE
SetPortVal KBC_KEY_DATA, btScancode, 1 '写入按键信息,按下键
End Sub
Sub MyKeyUpEx(ByVal vKeyCoad As Long) '模拟扩展键弹起
Dim btScancode As Long
btScancode = MapVirtualKey(vKeyCoad, 0)

KBCWait4IBE '等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
KBCWait4IBE
SetPortVal KBC_KEY_DATA, &HE0, 1 '写入扩展键标志信息
KBCWait4IBE '等待键盘缓冲区为空
SetPortVal KBC_KEY_CMD, &HD2, 1 '发送键盘写入命令
KBCWait4IBE
SetPortVal KBC_KEY_DATA, (btScancode Or &H80), 1 '写入按键信息，释放键
End Sub
```

还应该注意的是，如果要从扩展键转换到普通键，那么普通键的KeyDown事件应该发送两次。也就是说，如果我想模拟先按下一个扩展键，再按下一个普通键，那么就应该向端口发送两次该普通键被按下的信息。比如，我想模拟先按下左方向键，再按下空格键这个事件，由于左方向键是扩展键，空格键是普通键，那么流程就应该是这样的：

```vb
MyKeyDownEx VK_LEFT '按下左方向键
Sleep 200 '延时200毫秒
MyKeyUpEx VK_LEFT '释放左方向键

Sleep 500
MyKeyDown VK_SPACE '按下空格键，注意要发送两次
MyKeyDown VK_SPACE
Sleep 200
MyKeyUp VK_SPACE '释放空格键
```

好了，相信到这里，你的模拟按键程序也就差不多了，测试一下，是不是很有效呢，嘿嘿~~~~
WINIO组件的下载地址：http://www.114vip.com.cn/download/winio.zip
4.骨灰级模拟
方法3算是很底层的模拟了，我现在还没有发现有它模拟无效的程序。但是如果你用尽上面所有的方法，仍然无效的话，那么还有最后一个方法，绝对对任何程序都会有效，那就是：把键盘拿出来，老老实实地按下去吧。~~~~

嗯，终于写完了，休息一下~~~~~~~~~~~另外，由于水平所限，我说的这些或许会有错误的地方，希望大家多多指教。如果你有什么问题，也可以联系我，qq:511795070

以下是用winio模拟鼠标的函数，但是并不是在所有系统中都能正常运行，有需要的试试看吧

```vb
Sub MyMouseKey(MouseFun As Long, MyMouseX As Long, MyMouseY As Long, MyMouseZ As Long)
' 左键按下(MouseFun=9)。MyMouseX、MyMouseY、MyMouseZ 为0
' 右键按下(MouseFun=10)。MyMouseX、MyMouseY、MyMouseZ 为0
' 中键按下(MouseFun=12)。MyMouseX、MyMouseY、MyMouseZ 为0
' 任意键放开(MouseFun=8)。MyMouseX、MyMouseY、MyMouseZ 为0
' ------------------------------------
' 鼠标上移(MouseFun=8)。MyMouseY为移动距离，最大为255，最小为1。MyMouseX、MyMouseZ 为0
' 鼠标下移(MouseFun=40)。MyMouseY为移动距离，最大为1，最小为255。MyMouseX、MyMouseZ 为0
' 鼠标左移(MouseFun=24)。MyMouseX为移动距离，最大为1，最小为255。MyMouseY、MyMouseZ 为0
' 鼠标右移(MouseFun=8)。MyMouseX为移动距离，最大为255，最小为1。MyMouseY、MyMouseZ 为0
' ------------------------------------
KBCWait4IBE '等待缓冲区为空
SetPortVal 100, 211, 1 '发送鼠标写入命令
KBCWait4IBE '等待缓冲区为空
SetPortVal 96, MouseFun, 1 '发送鼠标动作命令
'-------------------------------------
KBCWait4IBE '等待缓冲区为空
SetPortVal 100, 211, 1 '发送鼠标写入命令
KBCWait4IBE '等待缓冲区为空
SetPortVal 96, MyMouseX, 1 '发送鼠标动作命令
'-------------------------------------
KBCWait4IBE '等待缓冲区为空
SetPortVal 100, 211, 1 '发送鼠标写入命令
KBCWait4IBE '等待缓冲区为空
SetPortVal 96, MyMouseY, 1 '发送鼠标动作命令
'-------------------------------------
KBCWait4IBE '等待缓冲区为空
SetPortVal 100, 211, 1 '发送鼠标写入命令
KBCWait4IBE '等待缓冲区为空
SetPortVal 96, MyMouseZ, 1 '发送鼠标动作命令
End Sub
```

示例：

```vb
MyMouseKey 9, 0, 0, 0 '左键按下
MyMouseKey 8, 0, 0, 0 '左键放开
MyMouseKey 10, 0, 0, 0 '右键按下
MyMouseKey 8, 0, 0, 0 '右键放开
MyMouseKey 12, 0, 0, 0 '中键按下
MyMouseKey 8, 0, 0, 0 '中键放开
MyMouseKey 8, 0, 5, 0 '上移5象素
MyMouseKey 40, 0,(255 xor 5),0 '下移5象素
MyMouseKey 24,(255 xor 5), 0, 0 '左移5象素
MyMouseKey 8, 5, 0, 0 '右移5象素
```

