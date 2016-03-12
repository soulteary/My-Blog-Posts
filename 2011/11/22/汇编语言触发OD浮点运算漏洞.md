# 汇编语言触发OD浮点运算漏洞

因为学习调试的时候,发现总是提示不准确的浮点数字.然后就猜了一下,如果调试的时候遇到不能调试的浮点数字,然后编译的时候又用编译器忽略的话,是不是可以用来保护软件.百度了一下,果不出所料,不过资料有点老,07年,不保证时效性,但是还是有价值的.

## 摘录如下

Themida壳利用了OllyDBG未公开的一个浮点指令漏洞来使得OllyDBG崩溃
Themida/WinLicense 最近一直很努力
自从Themida/WinLicense.V1.1.1.0放弃驱动后，Themida/WinLicense的Anti仿佛一下子就没了。
但是自Themida/WinLicense.V1.8.0.0，Oreans Technologies新增了HeapMagic检测，这个Anti方法目前还未见公开。
Themida/WinLicense.V1.8.2.0，Oreans Technologies利用了OllyDBG未公开的一个浮点指令漏洞来使得OllyDBG崩溃。

下面借助例子来简单说明一下Themida/WinLicense.V1.8.2.0利用的这个OllyDBG漏洞

MorGain结构快速设计安装程序.V2006.10.Revison.1571
http://www.morgain.com/download.htm
没发现其他新版加壳的小例子，所以借用这个
设置OllyDBG暂停在WinMain，载入HiDesign.exe，OllyDBG莫名其妙就自动退出了
设置OllyDBG暂停在系统断点，可以载入了，但是跟随入口后OllyDBG又自动退出了

可以用Hiew等工具观察一下HiDesign.exe的EP处代码

CODE:
00740014 B8 00000000 mov eax,0
00740019 60 pushad
0074001A 0BC0 or eax,eax
0074001C 74 68 je short 00740086
0074001E E8 00000000 call 00740023
00740023 58 pop eax
00740024 05 53000000 add eax,53
00740029 8038 E9 cmp byte ptr ds:[eax],0E9
0074002C 75 13 jnz short 00740041
0074002E 61 popad
0074002F EB 45 jmp short 00740076
00740031 DB2D 37007400 fld tbyte ptr ds:[740037] ;“37007400”这个值不唯一，可以变。
00740037 FFFF ???
00740039 FFFF ???
0074003B FFFF ???
0074003D FFFF ???
0074003F 3D 40E80000 cmp eax,0E840
00740044 0000 add byte ptr ds:[eax],al
00740046 58 pop eax
00740047 25 00F0FFFF and eax,FFFFF000

00740041处其实是：
00740041 E8 00000000 call 00740046
00740046 58 pop eax
00740047 25 00F0FFFF and eax,FFFFF000
再看一下Themida/WinLicense以前版本的入口代码：

CODE:
007F8014 B8 00000000 mov eax,0
007F8019 60 pushad
007F801A 0BC0 or eax,eax
007F801C 74 58 je short 007F8076
007F801E E8 00000000 call 007F8023
007F8023 58 pop eax
007F8024 05 43000000 add eax,43
007F8029 8038 E9 cmp byte ptr ds:[eax],0E9
007F802C 75 03 jnz short 007F8031
007F802E 61 popad
007F802F EB 35 jmp short 007F8066
007F8031 E8 00000000 call 007F8036
007F8036 58 pop eax
007F8037 25 00F0FFFF and eax,FFFFF000
Themida/WinLicense.V1.8.2.0 入口代码与之前版本的多了以下部分：

CODE:
00740031 DB2D 37007400 fld tbyte ptr ds:[740037]
00740037 FFFF ???
00740039 FFFF ???
0074003B FFFF ???
0074003D FFFF ???
0074003F 3D 40E80000 cmp eax,0E840
00740031 DB2D 37007400 fld tbyte ptr ds:[740037]
注意[740037]处10个字节：

CODE:
00740037 FF FF FF FF FF FF FF FF 3D 40 
浮点数：9.2233720368547758080e+18
OllyDBG目前最新V1.10版本处理此浮点数会崩溃！

我们只要在代码中构造一个如此的浮点指令陷阱，当OllyDBG V1.10反汇编至此时就会中招而崩溃退出

QUOTE:
OllyDBG V1.10 反汇编浮点指令陷阱：

fld tbyte ptr ds:[XXXXXXXX]
在[XXXXXXXX]处写入FF FF FF FF FF FF FF FF 3D 40


## 解决方案：

1. 期待OllyDBG新版修正此bug
2. 在OllyDBG V1.10调试前使用其他16进制工具修改掉[XXXXXXXX]中FF FF FF FF FF FF FF FF 3D 40数据，把其中的某位数据修改为任一其他值即可
3. 设置OllyDBG暂停在系统断点，在数据窗口中Ctrl+G：XXXXXXXX处，修改[XXXXXXXX]中FF FF FF FF FF FF FF FF 3D 40中任一数据为其他值
4. 修改OllyDBG V1.10的反汇编引擎，避开此浮点指令陷阱

