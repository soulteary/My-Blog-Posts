# [VB]判断其他程序是否为Enable

原文引自CSDN:http://topic.csdn.net/u/20090514/08/1a979292-c160-417a-abc7-65a1d325d23e.html 方法A:通过样式判断

```vb
Option Explicit

Private Const GWL_STYLE   As Long = -16

Private Const WS_DISABLED As Long = &H8000000

Private Declare Function GetWindowLong _
                Lib "user32.dll" _
                Alias "GetWindowLongA" (ByVal hWnd As Long, _
                                        ByVal nIndex As Long) As Long

Function IsEnabled(ByVal hWnd As Long) As Boolean

    Dim lStyle As Long

    lStyle = GetWindowLong(hWnd, GWL_STYLE)
    IsEnabled = ((lStyle And WS_DISABLED) = 0)
End Function

```

方法B:现成API

```vb
Private Declare Function IsWindowEnabled Lib "user32" (ByVal hwnd As Long) As Long 

If IsWindowEnabled(lngHwnd) Then MsgBox "可用状态"
```

方法C:发送消息获取返回值

```vb
Dim TextEnableStatus As Boolean
TextEnableStatus = Not TextEnableStatus
Call SendMessage(文本框件的hwnd, WM_ENABLE, ByVal TextEnableStatus, 0&)
```


