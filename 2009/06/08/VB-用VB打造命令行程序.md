# [VB]用VB打造命令行程序

其实还有一种，不过那个完全是调用管道来操作cmd.exe。 

这篇是天极上的一篇老文，感觉也是从什么地方转载的... 

原文出处：http://dev.yesky.com/9/2652009_1.shtml 

<!-- more -->

```vb
Option Explicit

'' API函数声明
Private Declare Function AllocConsole Lib &quot;kernel32&quot; () As Long

Private Declare Function FreeConsole Lib &quot;kernel32&quot; () As Long

Private Declare Function GetStdHandle Lib &quot;kernel32&quot; (ByVal nStdHandle As Long) As Long

Private Declare Function ReadConsole _
Lib &quot;kernel32&quot; _
Alias &quot;ReadConsoleA&quot; (ByVal hConsoleInput As Long, _
ByVal lpBuffer As String, _
ByVal nNumberOfCharsToRead As Long, _
lpNumherOfCharsRead As Long, _
lpReserved As Any) As Long

Private Declare Function WriteConsole _
Lib &quot;kernel32&quot; _
Alias &quot;WriteConsoleA&quot; (ByVal hConsoleOutput As Long, _
ByVal lpBuffer As Any, _
ByVal nNumberOfCharsToWrite As Long, _
lpNumberOfCharsWritten As Long, _
lpReserved As Any) As Long

Private Declare Function SetConsoleMode _
Lib &quot;kernel32&quot; (ByVal hConsoleOutput As Long, _
dwMode As Long) As Long

Private Declare Function SetConsoleTitle _
Lib &quot;kernel32&quot; _
Alias &quot;SetConsoleTitleA&quot; (ByVal lpConsoleTitle As String) As Long

Private Declare Function SetConsoleTextAttribute _
Lib &quot;kernel32&quot; (ByVal hConsoleOutput As Long, _
ByVal wAttributes As Long) As Long

''定义API函数中用到的所有常量
''GetStdHandle函数的 nStdHandle参数的取值
Private Const STD_INPUT_HANDLE = -10&amp;

Private Const STD_OUTPUT_HANDLE = -11&amp;

Private Const STD_ERROR_HANDLE = -12&amp;

''SetConsoleTextAttribute函数的wAttributes参数的取值（按RGB方式组合）
Private Const FOREGROUND_bLUE = &amp;H1

Private Const FOREGROUND_GREEN = &amp;H2

Private Const FOREGROUND_RED = &amp;H4

Private Const FOREGROUND_INTENSITY = &amp;H8

Private Const BACKGROUND_BLUE = &amp;H10

Private Const BACKGROUND_GREEN = &amp;H20

Private Const BACKGROUND_RED = &amp;H40

Private Const BACKGROUND_INTENSITY = &amp;H80

''SetConsoleMode的输入模式
Private Const ENABLE_LINE_INPUT = &amp;H2

Private Const ENABLE_ECHO_INPUT = &amp;H4

Private Const ENABLE_MOUSE_INPUT = &amp;H10

Private Const ENABLE_PROCESSED_INPUT = &amp;H1

Private Const ENABLE_WINDOW_INPUT = &amp;H8

''SetConsoleMode的输出模式
Private Const ENABLE_PROCESSED_OUTPUT = &amp;H1

Private Const ENABLE_WRAP_AT_EOL_OUTPUT = &amp;H2

Private hConsoleIn As Long ''控制台窗口的 input handle

Private hConsoleOut As Long ''控制台窗口的output handle

Private hConsoleErr As Long ''控制台窗口的error handle

''主程序
Private Sub Main()

Dim szUserInput As String

AllocConsole ''创建 console window
SetConsoleTitle &quot;VB控制台应用程序&quot;
''设置console window的标题
''取得console window的三个句柄
hConsoleIn = GetStdHandle(STD_INPUT_HANDLE)
hConsoleOut = GetStdHandle(STD_OUTPUT_HANDLE)
hConsoleErr = GetStdHandle(STD_ERROR_HANDLE)
SetConsoleTextAttribute hConsoleOut, FOREGROUND_GREEN Or FOREGROUND_INTENSITY
''前景：亮绿；背景：黑
ConsolePrint &quot;What''s your name?&quot;
szUserInput = ConsoleRead()

If Not szUserInput = vbNullString Then
ConsolePrint &quot;Hello, &quot; &amp; szUserInput &amp; &quot;!&quot; &amp; vbCrLf
Else
ConsolePrint &quot;You don''t have a name?&quot; &amp; vbCrLf
End If

ConsolePrint vbCrLf &amp; &quot;Press enter to exit!&quot;
Call ConsoleRead
FreeConsole ''销毁 console window
End Sub

''程序中用到的子函数
Private Sub ConsolePrint(szOut As String)
WriteConsole hConsoleOut, szOut, Len(szOut), vbNull, vbNull
End Sub

Private Function ConsoleRead() As String

Dim sUserInput As String * 256

Call ReadConsole(hConsoleIn, sUserInput, Len(sUserInput), vbNull, vbNull)
''截掉字符串结尾的&amp;H00和回车、换行符
ConsoleRead = Left$(sUserInput, InStr(sUserInput, Chr$(0)) - 3)
End Function
```

