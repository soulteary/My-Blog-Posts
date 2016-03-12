# [vb]判断素数方法总结

从网上偶然发现，稍加修改为函数，以备后用[感谢[http://blog.ncuhome.cn/niko/Logs/2006/6/1/8513.html](http://blog.ncuhome.cn/niko/Logs/2006/6/1/8513.html)博主的收集] 方法A原理：不能被2 ~ n-1整除

```vb
Private Function Check(lngTmp As Long) As Boolean

Dim lngIndex As Long, lngMax As Long

lngMax = lngTmp - 1

For lngIndex = 2 To lngMax

If lngTmp Mod lngIndex = 0 Then
Check = False

Exit For

End If

Next lngIndex

If lngIndex >= lngTmp Then
Check = True
Else
Check = False
End If

End Function
```

方法B原理:不能被2 ~ Sqr(n)整除 写法A：

```vb
Private Function CheckA(lngTmp As Long) As Boolean

Dim lngIndex As Long, lngMax As Long

lngMax = Int(Sqr(lngTmp))

For lngIndex = 2 To lngMax

If lngTmp Mod lngIndex = 0 Then Exit For

Next

If lngIndex > lngMax Then
CheckA = True
Else
CheckA = False
End If

End Function
```

写法B：

```vb
Private Function CheckA(lngTmp As Long) As Boolean  

Dim lngIndex As Long, lngMax As Long  

lngIndex = 2: lngMax = Int(Sqr(n))   

    Do While lngIndex <= lngMax   

        If lngTmp Mod I = 0 Then Exit Do  
        lngIndex = lngIndex + 1   
    Loop  

    If lngIndex > lngMax Then  
        CheckA = True  
    Else  
        CheckA = False  
    End If  

End Function
```

