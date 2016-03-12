# [VB]排队问题

知道网友的排队代码，感觉不错。修改的符合自己的标准，收藏。 原题如下：

```text
N个人排队从第一个开始报数，报到a的人出列，继续往下报，报到最后一个再从头开始继续报，求原队列中哪一个人最后出列，N和a由用户输入。
比如有12345，5(N=5)个人以3(a=3)报数，顺序为：123，3出列，剩下1245。继续往后数，451，1出列，剩下245。往后数为245，5出列剩下24，再数三下为242，2出列，最后是4出列。
```

```vb
Option Explicit
Dim bPeople() As Integer
Private Sub Form_Load()
    Me.Show
    Me.AutoRedraw = True
    Dim bMax As Byte, bTurn As Byte, bTmp As Byte, bCur As Byte, bCur2 As Byte, strTmp As String
    bMax = Val(InputBox("排队人数?", "输入"))
    bTurn = Val(InputBox("报数?", "输入"))
    ReDim bPeople(bMax)
    Do While bTmp < bMax
        bCur = bCur + 1
        If bCur = bMax + 1 Then bCur = 1
        If bPeople(bCur) <> 1 Then bCur2 = bCur2 + 1
        If bCur2 = bTurn Then
            strTmp = strTmp & bCur & " "
            bTmp = bTmp + 1
            bCur2 = 0
            bPeople(bCur) = 1
        End If
    Loop
    Print strTmp
End Sub
```

