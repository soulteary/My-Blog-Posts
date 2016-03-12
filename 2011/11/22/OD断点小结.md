# OD断点小结

转载一篇老文,个人感觉,需要多练习各种风格的语音,来掌握逆向的感觉. 比如这段可以很直接的看出来是使用 &拼合字符串,vbaStrCat连接几段,然后挨着传入API.

```asm
00532D1A  lea     ecx, dword ptr [ebp-38]          ;  WITH objWinSock
00532D1D  call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
00532D23  mov     dword ptr [ebp-4], 18            ;  
00532D2A  push    00443BEC                         ;  字串1
00532D2F  push    00443BF8                         ; /.
00532D34  call    dword ptr [<&MSVBVM60.__vbaStrCa>; \__vbaStrCat
00532D3A  mov     edx, eax
00532D3C  lea     ecx, dword ptr [ebp-38]
00532D3F  call    dword ptr [<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
00532D45  push    eax
00532D46  push    00443C00                         ; /字串2
00532D4B  call    dword ptr [<&MSVBVM60.__vbaStrCa>; \__vbaStrCat
00532D51  mov     edx, eax
00532D53  lea     ecx, dword ptr [ebp-3C]
00532D56  call    dword ptr [<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
00532D5C  push    eax
00532D5D  push    00443BF8                         ; /.
00532D62  call    dword ptr [<&MSVBVM60.__vbaStrCa>; \__vbaStrCat
00532D68  mov     edx, eax
00532D6A  lea     ecx, dword ptr [ebp-40]
00532D6D  call    dword ptr [<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
00532D73  push    eax
00532D74  push    00443C14                         ; /字串3
00532D79  call    dword ptr [<&MSVBVM60.__vbaStrCa>; \__vbaStrCat
00532D7F  mov     dword ptr [ebp-58], eax
00532D82  mov     dword ptr [ebp-60], 8
00532D89  mov     dword ptr [ebp-88], 1A4D         ;  端口
00532D93  mov     dword ptr [ebp-90], 2
00532D9D  mov     eax, 10
```

API断点 
Ollydbg中一般下API中断的方法，有二种。 
1.在代码窗口中点鼠标右键，出现功能菜单。在[搜索]选择项下有〔当前模块的名称〕和〔全部模块的名称〕俩项，选择其中的一项就打开了程序调用 API的窗口，在这个窗口中选择你要跟踪的API函数名。双击这个函数就能到程序的调用地址处。然后用F2下中断。也可以在API窗口中选择需要跟踪的函 数点鼠标右键出现功能菜单，选择〔在每个参考设置断点〕。同样下了断点。                                      
快捷方式：Ctrl+N 
2.在 命令行窗口 中输入BPX   API函数名或者BP   API函数名 后回车。这时出现了所有调用这个函数的地址的窗口，在这个窗口中可以看到调用这个API函数的地址已改变了颜色。说明下好了断点。 
说明一下：BPX一般中断在程序调用API的地址处。BP会中断在API的写入地址处。二这有所不同，根据需要选择。 

优点：这种方法下的断点是针对每一个API函数的，所以具有明确的目的。 
缺点：关键的API函数不容易找到。所以有时下的断点没有作用。


常用断点(OD中) 

实在找不到断点可以试下面的方法：
bmsg handle wm_gettext 拦截注册码（handle为对应窗口的句柄）
bmsg handle wm_command 拦截OK按钮（handle为对应窗口的句柄）

拦截窗口：

bpx CreateWindow 创建窗口
bpx CreateWindowEx(A/W) 创建窗口
bpx ShowWindow 显示窗口
bpx UpdateWindow 更新窗口
bpx GetWindowText(A/W) 获取窗口文本

拦截消息框：

bpx MessageBox(A) 创建消息框
bpx MessageBoxExA 创建消息框
bpx MessageBoxIndirect(A) 创建定制消息框

拦截警告声：

bpx MessageBeep 发出系统警告声(如果没有声卡就直接驱动系统喇叭发声)

拦截对话框：

bpx DialogBox 创建模态对话框
bpx DialogBoxParam(A/W) 创建模态对话框
bpx DialogBoxIndirect 创建模态对话框
bpx DialogBoxIndirectParam(A/W) 创建模态对话框
bpx CreateDialog 创建非模态对话框
bpx CreateDialogParam(A) 创建非模态对话框
bpx CreateDialogIndirect 创建非模态对话框
bpx CreateDialogIndirectParam(A/W) 创建非模态对话框
bpx GetDlgItemText(A/W) 获取对话框文本
bpx GetDlgItemInt 获取对话框整数值

拦截剪贴板：

bpx GetClipboardData 获取剪贴板数据

拦截注册表：

bpx RegOpenKey(A) 打开子健 ( 例：bpx RegOpenKey(A) if *(esp+8)=='****' )
bpx RegOpenKeyEx 打开子健 ( 例：bpx RegOpenKeyEx if *(esp+8)=='****' )
bpx RegQueryValue(A) 查找子健 ( 例：bpx RegQueryValue(A) if *(esp+8)=='****' )
bpx RegQueryValueEx 查找子健 ( 例：bpx RegQueryValueEx if *(esp+8)=='****' )
bpx RegSetValue(A) 设置子健 ( 例：bpx RegSetValue(A) if *(esp+8)=='****' )
bpx RegSetValueEx(A) 设置子健 ( 例：bpx RegSetValueEx(A) if *(esp+8)=='****' )
注意:“****”为指定子键名的前4个字符，如子键为“Regcode”，则“****”= “Regc”

==================

功能限制拦截断点：

bpx EnableMenuItem 禁止或允许菜单项
bpx EnableWindow 禁止或允许窗口
bmsg hMenu wm_command 拦截菜单按键事件，其中hMenu为菜单句柄
bpx K32Thk1632Prolog 配合bmsg hMenu wm_command使用，可以通过这个断点进入菜单处理程序 
拦截时间：

bpx GetLocalTime 获取本地时间
bpx GetSystemTime 获取系统时间
bpx GetFileTime 获取文件时间
bpx GetTickCount 获得自系统成功启动以来所经历的毫秒数
bpx GetCurrentTime 获取当前时间（16位）
bpx SetTimer 创建定时器
bpx TimerProc 定时器超时回调函数

拦截文件：

bpx CreateFileA 创建或打开文件 (32位)
bpx OpenFile 打开文件 (32位)
bpx ReadFile 读文件 (32位)
bpx WriteFile 写文件 (32位)
bpx _lcreat 创建或打开文件 (16位)
bpx _lopen 打开文件 (16位)
bpx _lread 读文件 (16位)
bpx _lwrite 写文件 (16位)
bpx _hread 读文件 (16位)
bpx _hwrite 写文件 (16位)

拦截驱动器：

bpx GetDrivetype(A/W) 获取磁盘驱动器类型
bpx GetLogicalDrives 获取逻辑驱动器符号
bpx GetLogicalDriveStringsA(W) 获取当前所有逻辑驱动器的根驱动器路径

拦截狗：

bpio -h 378(或278、3BC) R 378、278、3BC是并行打印端口
bpio -h 3F8(或2F8、3E8、2E8) R 3F8、2F8、3E8、2E8是串行端口

+++++++++++VB程序专用断点：++++++++++

bp__vbaFreeStr 偶发现了VB杀手断点.不管是重起验证.还是有错误提示的VB..下这个断点通杀

bpx msvbvm50!__vbaStrCmp 比较字符串是否相等
bpx msvbvm50!__vbaStrComp 比较字符串是否相等
bpx msvbvm50!__vbaVarTstNe 比较变量是否不相等
bpx msvbvm50!__vbaVarTstEq 比较变量是否相等
bpx msvbvm50!__vbaStrCopy 复制字符串
bpx msvbvm50!__vbaStrMove 移动字符串
bpx MultiByteToWideChar ANSI字符串转换成Unicode字符串
bpx WideCharToMultiByte Unicode字符串转换成ANSI字符串
上面的断点对应VB5程序，如果是VB6程序则将msvbvm50改成msvbvm60即可

VB程序的破解

VB程序使很多朋友感到头痛，主要是VB程序反编译时产生大量的垃圾代码，而且也找不到有
用的信息，在动态调试过程中，垃圾代码太多，往往迷失于冗余的代码中，找不到方向。　　　记住VB常用的

一些函数：
MultiByteToWideChar 将ANSI字符串转换成UNICODE字符
WideCHatToMultiByte　　将UNICODE字符转换成ANSI字符
rtcT8ValFromBstr　　　 把字符转换成浮点数　　
vbaStrCmp　　　　　　　 比较字符串（常用断点）
vbaStrComp　　　　　　 字符串比较（常用断点）
vbaStrCopy　　　　　　 复制字符串
StrConv　　　　　　　　转换字符串
vbaStrMove　　　　　　 移动字符串
__vbaVarCat 连接字符串
rtcMidCharVar 在字符串中取字符或者字符串!
__vbaLenBstr 取字符串的长度
vbaVarTstNe　　　　　　变量比较
vbaVarTstEq　　　　　　变量比较
rtcMsgBox　　　　　　　显示对话框
VarBstrCmp　　　　　　 比较字符串
VarCyCmp　　　　　　　 比较字符串
　　
用OD载入脱壳后的程序，在命令行输入：bpx hmemcpy，然后回车，会弹出程序运行调用的所有的函数，在每个

函数上设置好断点！说明：我破VB程序喜欢用这个断点设置方法，通过一步步跟踪，基本可以把握程序保护的

思路，所以我破VB程序基本用这个断点，当然你可以用其它的断点，只要能找到关键，任何断点都是用意义的

。

关于VB的程序，注册没有提示的二个办法：
第一（提示错误）：用GetVBRes来替换里面的提示串，一般是以‘111111’，‘222222’之类的替换
因为：VB，用的字来存放提示还有加了点东东，我们用的工具一般是字节分析。换成‘22222’之类的就是字节

了，用静态分析，就有你该的串了。GetVBRes（网上很多，自己下吧）

第二（没有提示）：用vbde这个工具（不知道，有没有用过DEDE，是一样思路），主要是找出破解的按钮窗口

的位置，来进行跟踪。

先给出修改能正确反编译VB程序的W32DASM的地址：
======================
offsets 0x16B6C-0x16B6D

修改机器码为： 98 F4
======================

VB程序的跟踪断点：

============
MultiByteToWideChar,
rtcR8ValFromBstr,
WideCharToMultiByte,
__vbaStrCmp
__vbaStrComp
__vbaStrCopy
__vbaStrMove
__vbaVarTstNe
rtcBeep
rtcGetPresentDate (时间API)
rtcMsgBox
=========

时间限制断点：

================
CompareFileTime
GetLocalTime
GetSystemTime
GetTimeZoneInformation
msvcrt.diffTime()
msvcrt.Time()
================

VB断点查找方法

1，VB6.0编写，OD载入程序调出注册窗口，alt+e调出可执行模块窗口找到X:\WINDOWS\system32\MSVBVM60.DLL
双击，在ctrl+n调出窗口找到，名称XXXXXXE区段=ENGINE 导出__vbaVarMove双击来到下面地址(可以直接在命

令行 bp __vbaVarMove)
回到程序注册窗口点注册被拦断在刚才下断的地址，断后在ctrl+F9，F8回
2，OD载入程序，命令行下断点。
bp rtcMsgBox
堆栈友好提示
确定注册失败按钮返回。接着向上找出点注册按钮执行的代码第一句，可以吗？当然行，根据我们知道程序员

写一个事件执行的代码是如这种，
各种语言都差不多。
3，OD载入程序，命令行下断点。
bp rtcMsgBox
任意填入伪注册码 9999999999999999999
确定后中断
堆栈友好提示
确定注册失败按钮返回。
W32Dasm反汇编程序，Shiht+F12
4，VB中的messagebox是一个消息框,汇编中用rtcMsgBox下断点.用olldbg载入程序,Alt+e,在可执行文件模块中

找到Msvbvm60.dll,双击它,
在代码窗口点右键-搜索-当前模块中的名称中的rtcMsgBox函数,双击它,在6A362F29 55 PUSH EBP这一句双击下

断点,关掉多余的窗口,只留下
cpu调试主窗口,F9运行程序,点?号按钮,随便输入987654321后,回车后立即中断,然后Ctrt+f9执行到返回地址,

因为这是msvbvm60的领空,
我们要回到程序领空.秘密记事本弹出message错误提示信息,点确定,向上看 ,再按F8就回到
5，为Microsoft Visual Basic 6.0。先用SmartCheck找到程序比较注册码点，
6，用vb常用比较断点
vbastrcmp
vbastrcomp
vbavartsteq
在od中设断点找注册码
7，用Od载入程序，运行，填入上面的注册码和顺序号。在Od中下断点,Alt+E,双击Msvbvm60运行库，右键－搜

索当前模块中的名称，找到Vbastrcmp，双击下断点。

--------------------------

注意：VB程序仍然可以使用普通API函数，只要函数“最终”CALL了这个函数
上面的断点对应VB6程序，如果是VB5程序则将msvbvm60改成msvbvm50即可

★注意：上面所列函数末尾有带“A”的，有带“W”的，有不带后缀的；一般说来，如果函数同时可以有后缀

也可以没有后缀（形如：MessageBox(A/W)）， 则不带后缀的表示16位的函数（MessageBox），带后缀的

（MessageBoxA、MessageBoxW）表示32位的函数；通常优先使用带后缀（A或W）的断点，带A后缀的一般用于

WIN9X系统， 而带W后缀的一般用于NT系统；如果函数没有任何后缀，则表示这是个通用的跨平台的API函数

