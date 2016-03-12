# [VB]随机洗牌的实现

[VB洗牌问题](http://zhidao.baidu.com/question/96996904.html) 如果把纸牌以二维数组的形式存储为a（1 to 4，1 to 13），怎样才能实现随机发牌？ 将牌随机发送到一维数组a（1 to 52）里面？ 我的解答，个人感觉已经极限了。不知道还有啥解法没。

```vb
Option Explicit

Dim bArr(1 To 13) As Byte

Dim bPoint        As Byte
'write by firendless
'PromiseForever.Com
Private Sub cmdGet_Click()
    Call GenNums

    Dim Cards(1 To 4, 1 To 13)

    Dim bType As Byte

    For bType = 1 To 4

        For bPoint = 1 To 13

            Cards(bType, bPoint) = bArr(bPoint)

            Debug.Print Cards(bType, bPoint)
            txtEcho(bType).Text = txtEcho(bType).Text & Cards(bType, bPoint) & ","
        Next

        Call GenNums
    Next

End Sub

Private Sub GenNums()

    Dim bIndex As Byte, bNum As Byte, bTmp As Byte

    For bIndex = 1 To 13

        bArr(bIndex) = bIndex

    Next

    For bIndex = 1 To 13

        Randomize
        bNum = Int(Rnd * (13 - bIndex + 1)) + bIndex

        bTmp = bArr(bIndex)
        bArr(bIndex) = bArr(bNum)
        bArr(bNum) = bTmp

    Next

End Sub

```

