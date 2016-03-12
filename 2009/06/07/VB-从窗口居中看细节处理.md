# [VB]从窗口居中看细节处理

其实多敲敲code后自然会有什么方式的代码最快最安全，其实设置居中真的很简单，但是差异还是很大的，如果你有兴趣可以点击阅读全文。

<!-- more -->

这个也是偶尔在网上看东西发现的小问题，其实也不算问题，不过如果目标机器资源紧张的时候，会出现窗口的一闪而过（改变位置。）
Move方法和这个是一样的，所以就忽略了。[通用的]
我们常常看到的设置窗体居中的方式有：
```vb
Me.Left = (Screen.Width - Me.Width) / 2
Me.Top = (Screen.Height - Me.Height) / 2
```

为了简单一点，我们或许会写成

```vb
With Me
    .Left = (Screen.Width - .Width) / 2
    .Top = (Screen.Height - .Height) / 2
End With
```

如果要通用我们或许会写作

```vb
Public Sub PutCenter(frm As Form)

With frm
    .Left = (Screen.Width - .Width) / 2
    .Top = (Screen.Height - .Height) / 2
End With

End Sub
```

然后可以在任意事件中调用 call putcenter(窗体Name) 但是这些都太复杂了，而且不安全，因为除法要考虑除数不为零，在一些情况下，或许的Screen会为零的哦~ 于是我们继续改，为了直观，我先不优化到底，保留着代数式的非最简结果吧。

```vb
Public Sub PutCenter(frm As Form)

With frm
    .Left = (Screen.Width - .Width) * (1/2)
    .Top = (Screen.Height - .Height) * (1/2)
End With

End Sub
```

这样的话，就把除数为零的隐患解决了，贯彻防御性编程的策略，我们还得考虑Screen返回数值为0的情况， 那么我们继续改~

```vb
Public Sub PutCenter(frm As Form)

Dim lngRet As Long

With frm
    lngRet = Abs(Screen.Width - .Width) * 0.5
    .Left = lngRet
    lngRet = Abs(Screen.Height - .Height) * 0.5
    .Top = lngRet
End With

End Sub
```

这段代码比起上面的好在于可以解决如果获取窗口本身Width和Height或者Screen的Width和Height为零后的问题。
而且帮助计算机归纳了1/2这步。

或许你说我的考虑没必要，但是请想想，是不是所有可能出错的事情终究都会出错呢？为了安全，还是谨慎点好~

如果你非要说设置窗口属性的StartUpPosition也可以实现居中，我只能对转牛角尖的您说：StartUpPosition只能在窗口初始化时使用，这个是只读属性，不能再次调用。除非你重新载入窗口，但是重载的代价是不是太大了呢？

