# [VB]使用VB操作EXCEL内的数据

本例是转换Excel中的A1中的数字为大写并保存到A2中。

```vb
Private Sub cmdGet_Click()
Dim lngTmp As Long
Dim xlsApp As Object
Set xlsApp = Excel.Application
With xlsApp
.Visible = True
.Workbooks.Open (App.Path & "\target.xls")
lngTmp = .Workbooks("target").Sheets("Sheet1").Range("A1").Value
.Workbooks("target").Close
.Quit
End With
Set xlsApp = Nothing
MsgBox "表格中的数字是：" & lngTmp & vbNewLine & "转换后的数字是：" & MyConvert(CStr(lngTmp))

End Sub

Private Function MyConvert(strNum As String) As String
Dim intLen As Integer, intIndex As Integer
Dim strTmp As String, strResult As String
intLen = Len(strNum): strResult = ""
For intIndex = 1 To intLen
strTmp = Mid(strNum, intIndex, 1)
strTmp = ConvertHan(strTmp)
strResult = strResult & strTmp
Next
MyConvert = strResult
End Function

Private Function ConvertHan(strNum As String) As String
Select Case strNum
Case Is = "0": ConvertHan = "零"
Case Is = "1": ConvertHan = "壹"
Case Is = "2": ConvertHan = "贰"
Case Is = "3": ConvertHan = "叁"
Case Is = "4": ConvertHan = "肆"
Case Is = "5": ConvertHan = "伍"
Case Is = "6": ConvertHan = "陆"
Case Is = "7": ConvertHan = "柒"
Case Is = "8": ConvertHan = "捌"
Case Is = "9": ConvertHan = "玖"
End Select
End Function
```

