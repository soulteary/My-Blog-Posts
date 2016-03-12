# [vb]网页编码函数

这个小工具中便使用了这个例子：[http://www.promiseforever.com/blog/show-437-1.html](http://www.promiseforever.com/blog/show-437-1.html) 发一段一直在用的网页编码函数，希望能帮得上有需要的人。 

我们常常看到百度Google搜索的地址中出现一堆%E8%8A%B1%E5%A5%BD%E6%9C%88%E5%9C%86之类的代码。 

这些代码其实是被转义后的汉字，使用下面的函数可以轻松的将汉字转换为这种代码。

<!-- more -->

```vb
Private Function UTF8EncodeURI(strTmp As String) As String  

    Dim lngLen  As Long, lngAsc As Long, lngIndex As Long  

    Dim strChar As String, strUni As String, strResult As String  

    If strTmp = "" Then  
        UTF8EncodeURI = strTmp   

        Exit Function  

    End If  

    lngLen = Len(strTmp)   

    For lngIndex = 1 To lngLen   
        strChar = Mid$(strTmp, lngIndex, 1)   
        lngAsc = AscW(strChar)   

        If lngAsc < 0 Then lngAsc = lngAsc + 65536   

        If (lngAsc And &HFF80) = 0 Then  
            strResult = strResult & strChar   
        Else  

            If (lngAsc And &HF000) = 0 Then  
                strUni = "%" & Hex$(((lngAsc  2 ^ 6)) Or &HC0) & Hex$(lngAsc And &H3F Or &H80)   
                strResult = strResult & strUni   
            Else  
                strUni = "%" & Hex$((lngAsc  2 ^ 12) Or &HE0) & "%" & Hex$((lngAsc  2 ^ 6) And &H3F Or &H80) & "%" & Hex$(lngAsc And &H3F Or &H80)   
                strResult = strResult & strUni   
            End If  
        End If  

    Next  

    UTF8EncodeURI = strResult   
End Function  

Private Function GBKEncodeURI(strTmp As String) As String  

    Dim bData()   As Byte  

    Dim strResult As String  

    Dim lngUbound As Long, lngLbound As Long, lngIndex As Long  

    strResult = ""  

    bData = StrConv(strTmp, vbFromUnicode)   
    lngUbound = UBound(bData)   
    lngLbound = LBound(bData)   

    For lngIndex = lngLbound To lngUbound   
        strResult = strResult & "%" & Hex$(bData(lngIndex))   
    Next  

    GBKEncodeURI = strResult   
End Function  
```

