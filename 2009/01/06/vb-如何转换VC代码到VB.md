# [vb]如何转换VC代码到VB

<!-- more -->

1.定义常量

我们首先看第一个例子:

```c
#define STD_COPY1//COMMCTRL.H  
```

在VC 代码中用#define定义常量,而在VB中是用Const来定义,因此转换成VB的代码是:

```vb
Public Const STD_COPY=1   
```

第二个例子:
```c
#define LB_SELECTSTRING0x018C//WINUSER.h  
```

这有一个问题,VC 中所有以"0x"开头的值是十六进制,而在VB中表示十六进制要用"&H"开头.因此转换成VB的代码为:

```vb
Const LB_SELECTSTRING=0x018C
```

第三个例子:

```c
#define TCN_FIRST(0U-550U)  
```

这里有个特别的是定义的值是以"U"结尾,这是意味着该常量的类型是"unsigned long"(在VB中不支持该数据类型).但是unsignedlong和signedlong(在VB中是Long)两种数据类型的值在存储方式上是一样的,只是表现的形式不同.因此,我们只需要去掉末尾的"U"就可以了.

```vb
Const TCN_FIRST=(0-550)  
```

这同样适合以"L"结尾的值。

2.结构的定义

我们先看VC 定义的一个比较简单的结构:

```c
type defstruct tagTBSAVEPARAMSA{   
HKEYhkr;   
LPCSTRpszSubKey;   
LPCSTRpszValueName;   
}TBSAVEPARAMSA,FAR*LPTBSAVEPARAMSA;  
```

首先我们需要把第一行的"type defstruct"转换成"Public Type"


```vb
Public Type tagTBSAVEPARAMSA    
```

然后处理结构成员,

```c
HKEYhkr;    
LPCSTRpszSubKey;    
LPCSTRpszValueName;  
```

对于第一个成员类型HKEY.我们要知道VC 中的以"H"开头的大部分数据类型代表的是某种句柄.在VB中每一个Form对象和许多控件都有一个hWnd属性,它代表所属窗口的句柄.hWnd的类型是Long,并且所有用来存储句柄的变量类型都是Long.因此,该成员在VB中定义为:

```vb
hkr As Long
```

同样的,VC 数据类型LPSTR和LPCSTR代表指向字符串的指针,在VB中可以当作String类型.因为当你传送结构给API时,VB将把结构中所有的String转换成指向ANSI字符串的指针.因此后两个成员在VB中表示为:

```vb
pszSubKey As String 
pszValueName As String
```

对于最后一行"}TBSAVEPARAMSA,FAR*LPTBSAVEPARAMSA;"我们只需要用" End Type "取代

就可以了.转换成VB代码后完整的结构为:

```vb
Public Type tagTBSAVEPARAMSA    
hkr As Long    
pszSubKey As String    
pszValueName As String    
End Type 
```

以下是VC 中数据类型对应到VB中的数据类型VC 数据类型VB数据类型


```c
short					Integer    
int					Long    
long					Long    
UNIT					Long    
ULONG					Long    
WORD,DWORD					Long    
WPARAM,LPARAM					Long    
WMSG,UMSG					Long    
HRESULT					Long    
BOOL					Boolean    
COLORREF					Long    
HWND,HDC,HBRUSH,HKEY,等等.					Long
LPSTR,LPCSTR					String
LPWSTR,OLECHAR,BSTR					String
LPTSTR					String
VARIANT_BOOL					Boolean
unsignedchar					Byte
BYTE					Byte
VARIANT					Variant
(任何以*或**结尾的数据类型)					Long
```

3.函数的转换

我们知道VB提供了APIVieweradd-in,但是有很多API函数它并没有包括在内. 因此知道如何把VC 函数转换成VB的函数格式是非常重要的.先看第一个例子:

```c
WINCOMMCTRLAPIHWNDWINAPI    
CreateStatusWindowsA(LONGstyle,    
LPCSTRlpszTest,HWNDhwndParent,UINTwID);   
```

这个函数创建一个StatusBar控件.从WINCOMMCTRLAPI可以得知该函数来自动态链接库ComCtl32.dll.(有时,我们需要从MSDN中查找某函数对应的DLL)然后我们知道该函数的返回类型是HWND,对应VB的类型是Long.最后,根据前面提到类型对应表,很容易的转换相应的函数参数.

```vb
Public Declare Function CreateStatusWindowA Lib "ComCtl32.dll" (Byvalstyle As Long, ByvallpszText As String, ByvalhwndParent As Long,ByvalwID As Long) As Long  
```


