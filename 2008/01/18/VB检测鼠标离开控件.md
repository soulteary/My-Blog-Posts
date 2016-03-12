# VB检测鼠标离开控件

VB 提供了3种很棒的鼠标交互事件：MouseDown、MouseUp、MouseMove，但是唯独缺少一个MouseExit（鼠标移出）事件。然而这个缺少的事件却是我们的刚需。

为了解决问题，我们可以通过API来自己封装一个离开事件。

首先引入两个API声明：

```vb
Private Declare Function SetCapture Lib "user32" (ByVal hWnd As Long) As Long
Private Declare Function ReleaseCapture Lib "user32" () As Long
```

然后，可以在控件（以 Picture1 为例）的 MouseMove 事件上加上以下代码：

```vb
Dim MouseExit As Boolean

MouseOver = (0 <= X) And (X <=Picture1.Width And (0 <=Y) And (Y <=Picture1.Height)

If MouseExit Then

	SetCapture Picture1.hWnd

Else

	ReleaseCapture

End If
```


