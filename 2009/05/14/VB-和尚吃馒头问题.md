# [VB]和尚吃馒头问题

和尚吃馒头的问题,俺恶搞一下。 大和尚一个人一个馒头，小和尚三个人吃一个馒头。话说寺庙的馒头也不算大呀。那么小和尚得多小。。。

```vb
Private Sub Form_Load()
'100个和尚吃100个馒头。大和尚一人吃3个，小和尚3人吃一个
Dim 大和尚人数 As Byte
Dim 小和尚人数 As Byte
For 大和尚人数 = 1 To 100 Step 1
For 小和尚人数 = 3 To 99 Step 3
If 大和尚人数 + 小和尚人数 = 100 Then Debug.Print "大和尚人数" & 大和尚人数 & " And 小和人数尚" & 小和尚人数
Next
Next
End Sub
```

