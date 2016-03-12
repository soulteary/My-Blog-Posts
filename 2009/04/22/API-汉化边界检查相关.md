# [API]汉化边界检查相关

原文blog.potatoneko.cn/?m=200812 从花瓣区位看破解CreateFontA区位 挖掘者ChinaAVG Shane007

首先，分析游戏的文字显示方式。玩游戏的时候最怕打开游戏看到一片乱码，现在汉化游戏却是最希望打开游戏看到一片乱码，因为一般在非日文环境下能正常显示日文的游戏都使用的字符集限定，为汉化增加了不少的难度……

一起采取Textout方式的年代，日文、繁体中文十个有九个是乱码，于是有了Apploc和NL，后来比尔大叔推出了CreateFontA函数……CreateFontA能够直接定义程序使用和操作系统不同的字符集，于是乱码没有了，而汉化者噩梦开始了，因为汉化后的中文内容在经过日文字符集的编译后变为日语状态下的乱码。

以下是比尔大叔的MSDN Library对CreateFontA给出的函数原形

```c
HFONT CreateFont(
int nHeight, // height of font
int nWidth, // average character width
int nEscapement, // angle of escapement
int nOrientation, // base-line orientation angle
int fnWeight, // font weight
DWORD fdwItalic, // italic attribute option
DWORD fdwUnderline, // underline attribute option
DWORD fdwStrikeOut, // strikeout attribute option
DWORD fdwCharSet, // character set identifier
DWORD fdwOutputPrecision, // output precision
DWORD fdwClipPrecision, // clipping precision
DWORD fdwQuality, // output quality
DWORD fdwPitchAndFamily, // pitch and family
LPCTSTR lpszFace // typeface name
);
  
HFONT CreateFontIndirect(
CONST LOGFONT* lplf // characteristics
);  
```

其中 LOGFONT的声明如下：

```c
typedef struct tagLOGFONT {
LONG lfHeight;
LONG lfWidth;
LONG lfEscapement;
LONG lfOrientation;
LONG lfWeight;
BYTE lfItalic;
BYTE lfUnderline;
BYTE lfStrikeOut;
BYTE lfCharSet;
BYTE lfOutPrecision;
BYTE lfClipPrecision;
BYTE lfQuality;
BYTE lfPitchAndFamily;
TCHAR lfFaceName[LF_FACESIZE];
} LOGFONT, *PLOGFONT;  
```

要改变程序支持的字符集，就要改变程序调用上面两个函数时的fdwCharSet或lfCharSet的值<!--more-->

其中各字符集所对应的值如下：

```text
字符集 值（十进制）
ANSI_CHARSET 0
DEFAULT_CHARSET 1
SYMBOL_CHARSET 2
MAC_CHARSET 77
SHIFTJI_CHARSET 128
HANGEUL_CHARSET 129
HANGUL_CHARSET 129
JOHAB_CHARSET 130
GB2312_CHARSET 134
CHINESEBIG5_CHARSET 136
GREEK_CHARSET 161
TURKISH_CHARSET 162
VIETNAMESE_CHARSET 163
HEBREW_CHARSET 177
ARABIC_CHARSET 178
BALTIC_CHARSET 186
RUSSIAN_CHARSET 204
THAI_CHARSET 222
EASTEUROPE_CHARSET 238
OEM_CHARSET 255
```

可以看到简体中文是86（Hex），日语是80（Hex），繁体中文是88（Hex）

我们要做的就是找到游戏中定义调用字符集的部位，将80改为86，这样就能让程序正确的显示中文。

用pexplorer打开HANABIRA.exe，进入反编汇模式，查找Font字符串，我们发现调用CreateFontA函数的位置是唯一的：

于是在以下代码我们停下来

```asm
L0041E365:

mov edi,[esp+14h]

mov ebx,[esp+28h]

mov ebp,[esp+24h]

mov eax,[esp+20h]

mov ecx,[esp+1Ch]

push edi

mov edx,[esp+30h]

push 00000031h

push 00000002h

push 00000000h

push 00000000h

push 00000080h

push ebx

push ebp

push eax

mov eax,[esp+3Ch]

push ecx

push 00000000h

push edx

push 00000000h

push eax

call [GDI32.dll!CreateFontA]

lea ecx,[esi+28h]

mov [esi+00000138h],eax

mov eax,edi

sub ecx,edi

lea esp,[esp+00h]
```

注意这就是调用GDI32.dll中的CreateFontA函数了，我们在这个堆栈中寻找将80这个值传递给CreateFontA的部位。

`push 00000080h`

就是这里将80值压入传递中

PE中标记了这段赋值的Hex数值，用UE打开文件，找到

`68800005355`

将其改为

`68860005355`

保存之。

这样PE中看到这段压入就成了

`push 00000086h`

初战告破，运行游戏，你会看到日文全变成了乱码，说明程序已经在使用中文的字符集了

修改游戏脚本，加入几个中文看看……

为什么我添加的中文全部是“□”？

这就是需要解决的第二关卡，字符集边界检查。

既然已经设定了字符集，为什么还要边界检查呢？这是为了防止当游戏文本中含有某些非法的字符串时产生缓存溢出。于是在字形传递到GDI32.dll描绘字体准备显示在屏幕之前对其进行检查，发现超出了设定的缓存大小就将其拦截下来，于是屏幕上就显示出一个“□”。我们知道，由于日文的字符比中文少得多，所以这个缓存也小的多，换言之就是边界太窄。

边界检查的例子：


```asm
cmp al,80

jbe xxxxxxxx

cmp al,09F

jb xxxxxxxx

cmp al,0E0

jb xxxxxxxx

cmp al,0FC

ja xxxxxxxx
```

即看字符是否在80-9F（前两位）和e0-fc（后两位）之间

===================================

如何具体查找程序的字符集边界呢？常用的方法是下断，用OllyDbg载入游戏主程序运行，一步一步断下去，在出现一堆“□”的时候停住，然后转到ASM模式查看停在哪里。


```asm
L0043BA00:

cmp al,80h

jc L0043BA08

cmp al,9Fh

jbe L0043BA10

L0043BA08:

cmp al,E0h

jc L0043BA30

cmp al,FFh

ja L0043BA30
```

花瓣的上边界为80~FF，而下边界为E0~FF，好，开始动手

这里我们将上边界改为80~FF，而下边界范围足够宽广，就不用改了。

这里为什么使用80~FF而不改为20~FF呢？因为我们需要让游戏文本中原本的日语空格（8140）不显示为乱码，于是8140刚好在边界外，就不会被送去CreateFontA进行显示，就会显示为日语的缺字码，一个空白——而它刚好就是空白，在显示上两者没有任何区别。

如法炮制，打开EU修改之，保存测试，OK，正常显示了

