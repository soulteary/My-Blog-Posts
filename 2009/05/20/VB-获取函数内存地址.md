# [VB]获取函数内存地址

类的回调出处为VBG高手代码，引入委托的概念。 

常规的AddressOf在类模块里不行是因为 

1. this指针 
2. HRESULT返回值 先贴获取API函数地址的代码

```vb
Option Explicit
Private Declare Function LoadLibrary Lib "kernel32" Alias "LoadLibraryA" (ByVal lpLibFileName As String) As LongPrivate Declare Function GetProcAddress Lib "kernel32" (ByVal hModule As Long,ByVal lpProcName As String) As Long
Public Function MyFunction(strDll As String, strFunction As String) As Long
 Dim lngRtn As Long, lngFunAddr As Long
 lngRtn = LoadLibrary(strDll)
 If lngRtn = 0 Then MsgBox "Err.": End
  lngFunAddr = GetProcAddress(lngRtn, strFunction)
   If lngFunAddr = 0 Then MsgBox "Err.": End
 MyFunction = lngFunAddrEnd FunctionPrivate Sub main()Dim lngRtn As Long: lngRtn = 0lngRtn = MyFunction("kernel32.dll", "Beep")MsgBox "函数地址为：" & lngRtn
End Sub
```

再贴获取自定义函数的代码

```vb
Option Explicit
'主函数Private Sub Main()

 Dim lngRtn As Long: lngRtn = 0

 lngRtn = FunctionAddress(AddressOf MyTarget)
 MsgBox "模块中的 MyTarget 函数地址为：" & lngRtn
End Sub'获取函数地址Private Function FunctionAddress(ByVal lngTmp As Long) As Long
 FunctionAddress = lngTmpEnd Function'要被获取地址的函数[函数恐慌的看着你...你..你...你要干嘛..- -!]
Private Function MyTarget() As Long
 MyTarget = 0
End Function
```

最后贴VBG高手的模拟委托的代码

```vb
很多api功能都需要用户编写回调函数。大家都知道只有模块(bas)里的函数才能使用addressof操作符，那么为了写回 调函数就必须建模块，这样可能破坏了你预想的代码结构(为了写一个函数就建一个模块?)而且还有种种的不便(有些多次使用的回调 是无法通过一个函数完成的)。
下面的代码告诉你如何解决这个问题。
下面这段写在模块MethodDelegate.bas里
------------------------------------------------------------ --
'''''''''''''''''''''''''''''''''' Copyright 2005 Newdraw '' MethodDelegate.bas ''? ; ;? ; ;? ; ; '' 用于回调类中的方法&am p;nb sp; ''''''''''''''''''''''''''''''''''
Option Explicit
Private Declare Sub CopyMemory Lib "kernel32" Alias "RtlMoveMemory" (Destination As Any, Source As Any, ByVal Length As Long)
'保存每个委托Private DelegateColl As New Collection

'委托代码被回调后使用该函数call类中函数'id 对应DelegateColl中的索引Private Sub CallBackDelegateProc(ByVal id As Long)With DelegateColl(id)If IsMissing(.Param) ThenInteraction.CallByName .Object, .Method, VbMethodElseInteraction.CallByName .Object, .Method, VbMethod, .ParamEnd IfEnd WithEnd Sub
'构造回调函数'id 对应DelegateColl中的索引Private Function MakeCallBackProc(id As Long) As Byte()
Dim code(12) As Byte
'push aabbccddcode(0) = &H68CopyMemory code(1), id, 4
'mov eax, aabbccddcode(5) = &HB8CopyMemory code(6), AddressOf CallBackDelegateProc, 4
'call eaxcode(10) = &HFFcode(11) = &HD0 'E0 .. jmp D0 .. call
'retcode(12) = &HC3
MakeCallBackProc = code
End Function
'新建委托'Object 被回调对象'Method 被回调方法名'Param 被回调方法参数,可略'返回 委托过程入口偏移地址Public Function NewDelegate(Object As Object, Method As String, Optional Param As Variant) As Long
With New DelegateDataSet .Object = Object.Method = MethodIf IsObject(Param) ThenSet .Param = ParamElse.Param = ParamEnd If.SetCallBackProc MakeCallBackProc(DelegateColl.Count + 1)
DelegateColl.Add .SelfNewDelegate = .CallBackProcEnd With
End Function------------------------------------------------------------ --
这段代码写在DelegateData.cls里
------------------------------------------------------------ --
'''''''''''''''''''''''''''''''''' Copyright 2005 Newdraw '' DelegateData.cls ''? ; ;? ; ;? ; ; '' 委托的数据 ''''''''''''''''''''''''''''''''''
Option Explicit
Public Object As ObjectPublic Method As StringPublic Param As VariantPublic CallBackProc As Long
Private CallBackCode() As Byte
Public Sub SetCallBackProc(code() As Byte)CallBackCode = codeCallBackProc = VarPtr(CallBackCode(0))End Sub
Public Property Get Self() As ObjectSet Self = MeEnd Property------------------------------------------------------------ --
下面是我写在窗体里的测试代码，你可以看到窗体的标题栏变成时钟了
------------------------------------------------------------ --
Private Declare Function SetTimer Lib "user32" (ByVal hwnd As Long, ByVal nIDEvent As Long, ByVal uElapse As Long, ByVal lpTimerFunc As Long) As LongPrivate Declare Function KillTimer Lib "user32" (ByVal hwnd As Long, ByVal nIDEvent As Long) As LongPrivate Const timer_id As Long = 5720
Public Sub TimerCallBack()Me.Caption = Time()End Sub
Private Sub Form_Load()SetTimer Me.hwnd, timer_id, 1000, MethodDelegate.NewDelegate(Me, "TimerCallBack")End Sub
Private Sub Form_Unload(Cancel As Integer)KillTimer Me.hwnd, timer_idEnd Sub------------------------------------------------------------ --
需要注意的是,这里的委托(Delegate)可不是.net里那样的安全机制，这仅仅是扩展vb的功能而已。使用的时候大家必 须谨慎,否则就是"非法操作"...
大家也看到了,这段代码还不完善. 一起来修改它吧!Edited by: publicsub
```

