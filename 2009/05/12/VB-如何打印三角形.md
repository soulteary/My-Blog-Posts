# [VB]如何打印三角形

百度知道的一道基础题，算法上面，如果考虑CPU执行时间，可以继续优化。俺做了2种出来，下面是两种算法的实现 第一种有优化和没优化的版本，第二种本身就是优化的，所以就没有啰嗦版本了。 5.12补充了网友的版本，贴在最下面。 [VB打印三角形](http://zhidao.baidu.com/question/96949583.html) 打印出以下图案：

```text
* 
* * * 
* * * * * 
* * * * * * * 
* * * * * 
* * * 
*
```

如果要优化的话，还可以更快，不过算法就不明晰了。 下面是未优化的版本

```vb
Option Explicit
Private Sub Form_Load()
    Me.AutoRedraw = True    '如果你要在窗体上绘制的话
    Const bLine As Byte = 7    '输出的行数
    Dim bIndex  As Byte, bMax As Byte, bSum As Byte, bMid As Byte
    bMid = bLine / 2    '获取中线
    bMax = 2 * bMid - 1    '获取中线字符数目
    '开始绘制
    For bIndex = 1 To bLine
        If bIndex <= bMid Then
            bSum = 2 * (bIndex - 1) + 1
        Else
            bSum = (2 * (bIndex - 1) + 1) - (4 * (bIndex - bMid))
        End If
        Debug.Print String$(bSum, "*")
        Me.Print String$(bSum, "*")
    Next

End Sub
```

接下来提速一下：

```vb
Option Explicit
Private Sub Form_Load()
    Me.AutoRedraw = True    '如果你要在窗体上绘制的话
    Const bLine As Byte = 7    '输出的行数
    Dim bIndex  As Byte, bMax As Byte, bSum As Byte, bMid As Byte
    bMid = bLine / 2    '获取中线
    bMax = 2 * bMid - 1    '获取中线字符数目
    '开始绘制
    For bIndex = 1 To bLine
        If bIndex <= bMid Then
            bSum = 2 * (bIndex - 1) + 1
        Else    '(2 * (bIndex - 1) + 1) - (4 * (bIndex - bMid))合并同类项
            bSum = 4 * bMid + (-2) * bIndex - 1
        End If
        Debug.Print String$(bSum, "*")
        Me.Print String$(bSum, "*")
    Next

End Sub

```

最后是在优化了一下的。

```vb
Option Explicit
Private Sub Form_Load()
    Me.AutoRedraw = True    '如果你要在窗体上绘制的话
    Const bLine As Byte = 7    '输出的行数
    Dim bIndex  As Byte, bMax As Byte, bSum As Byte, bMid As Byte, bTmp As Byte
    bTmp = bLine - 1
    bMid = bTmp / 2   '获取中线
    bMax = 2 * bMid - 1    '获取中线字符数目

    '开始绘制
    For bIndex = 0 To bTmp
        If bIndex <= bMid Then
            bSum = 2 * (bIndex) + 1
        Else    '2 * (bIndex) + 1 - (bIndex - 3) * 4合并同类项
            bSum = 13 - 2 * bIndex
        End If

        Debug.Print String$(bSum, "*")
        Me.Print String$(bSum, "*")
    Next

End Sub
```

有网友写出了简单实用的漂亮代码，俺用自己的规范格式化了，一下，贴上来留念。

```vb
Private Sub Form_Load()
    Me.AutoRedraw = True
    Dim intIndex As Integer, strTmp As String
    For intIndex = -6 To 6 Step 2
        strTmp = String(7 - Abs(intIndex), "*")
        Print strTmp
        Debug.Print strTmp
    Next intIndex
End Sub
```


