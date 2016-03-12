# [VB]混合字串的截断

[![demo-for-nc-text](https://attachment.soulteary.com/2011/07/13/demo-for-nc-text.png "demo-for-nc-text")](https://attachment.soulteary.com/2011/07/13/demo-for-nc-text.png)

<!-- more -->

混合字串概念上是指编码一致的中英或者说是Latin+GBK,Ascii+UniCode.
通俗的说就是中英混合字串的截断,当然GBK包含CJK(中日韩文字),通俗准确的说是单双字节字符混合字串。

其实用VS的话,可以引用SYSTEM.TEXT,然后使用encoding来直接操作字串,但是笔记本上实在跑不动VS2010...PHP的话,则可以直接用正则+mb_XXX函数来搞..
为了本上的PHP环境,为了本上刚刚安装的石器时代(自己玩的单机)..咬了咬牙,不安VS了,忍痛安装了VB6精简版(10MB)..
很久木有写VB CODE了..都快忘记鸟...起手居然写了" function FirCut(){..."汗.
下面的代码应该很好修改优化提高效率.不过应该已经够用了...

```vb
'MAX BYTES EACH LINE.
Private Const MAX_ONE_LINE As Byte = 23 * 2

Private Sub FirCut(strContent As String)

    Dim bLen    As Byte: bLen = 0

    Dim strTemp As String

    Do While LenB(StrConv(strContent, vbFromUnicode)) > 0

        strTemp = StrConv(MidB$(StrConv(strContent, vbFromUnicode), 1, MAX_ONE_LINE), vbUnicode)

        bLen = LenB(StrConv(strTemp, vbFromUnicode))

        'echo result by yourself
        lstEvent.AddItem strTemp

        If LenB(StrConv(strContent, vbFromUnicode)) >= MAX_ONE_LINE Then

            If AscB(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE, 1)) Then
                strContent = StrConv(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE + 1, LenB(StrConv(strContent, vbFromUnicode)) - 1), vbUnicode)
            Else
                strContent = StrConv(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE, LenB(StrConv(strContent, vbFromUnicode))), vbUnicode)
            End If

        Else

            Exit Do

        End If

    Loop

End Sub
```

话说为啥要写这个呢,想起来了,这个主要用于南茜Nancy18-19基于SHANE007的HOOK汉化工具的字符处理.
007大神搞定的那个工具似乎在这两款游戏上有点不支持,
他自己的话是这么说的:

> 有一个特别注意点，今天才发现的，文本每行最多为24个全角汉字（或48个半角英文），但是每行最后1个字符不能为全角汉字，就是说实际上文本每行最多为23个全角汉字，最后要加2个半角空格。如果不满23个全角汉字就要换行就要补相应的半角空格。否则游戏崩溃。一个画面最多可输出约92个汉字（在目前大字体的情况下）。 

所以，上面的代码就变成了下面的样子：

```vb
'每行允许汉字数目
Private Const MAX_ONE_LINE As Byte = 23 * 2
'23汉字后补助的空格数量
Private Const FIX_SPACES   As Byte = 1 * 2 

Private Sub FirCut(strContent As String)

    Dim bLen    As Byte: bLen = 0

    Dim strTemp As String

    Do While LenB(StrConv(strContent, vbFromUnicode)) > 0

        strTemp = StrConv(MidB$(StrConv(strContent, vbFromUnicode), 1, MAX_ONE_LINE), vbUnicode)

        bLen = LenB(StrConv(strTemp, vbFromUnicode))

        strTemp = strTemp & Space$(MAX_ONE_LINE - bLen) & Space$(FIX_SPACES)
'echo by yourself
        lstEvent.AddItem strTemp

        If LenB(StrConv(strContent, vbFromUnicode)) >= MAX_ONE_LINE Then

            If AscB(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE, 1)) Then
                strContent = StrConv(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE + 1, LenB(StrConv(strContent, vbFromUnicode)) - 1), vbUnicode)
            Else
                strContent = StrConv(MidB$(StrConv(strContent, vbFromUnicode), MAX_ONE_LINE, LenB(StrConv(strContent, vbFromUnicode))), vbUnicode)
            End If

        Else

            Exit Do

        End If

    Loop

End Sub
```

如果你看过代码，会发现没有对92个字符进行处理，也没有对如果换行的时候出现</n> 这类字符的情况出现处理。
是的，或许这个代码需要安装了这个游戏的人去试验，遇到那个问题需要怎么做，如果译文翻页数目大于原文翻页数目会怎么样等。
我没有想下载那个游戏，也知道完全木有精力去调试007留下的HOOK TOOLS.
所以,代码给出来了,哪位愿意继续做就接着呗。
最后上传一个没有进行92字长判断和是否截断分页的DEMO程序，谁想下载看是神马意思就下了玩玩。

文件下载:[download id="74"]


