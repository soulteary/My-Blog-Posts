# [vb]下载网页数据函数

```vb
Option Explicit    

Private Declare Function InternetOpen _    
                Lib "wininet" _    
                Alias "InternetOpenA" (ByVal sAgent As String, _    
                                       ByVal lAccessType As Long, _    
                                       ByVal sProxyName As String, _    
                                       ByVal sProxyBypass As String, _    
                                       ByVal lFlags As Long) As Long   

Private Declare Function InternetCloseHandle Lib "wininet" (ByRef hInet As Long) As Long   

Private Declare Function InternetReadFile _    
                Lib "wininet" (ByVal hFile As Long, _    
                               sBuffer As Byte, _    
                               ByVal lNumBytesToRead As Long, _    
                               lNumberOfBytesRead As Long) As Integer   

Private Declare Function InternetOpenUrl _    
                Lib "wininet" _    
                Alias "InternetOpenUrlA" (ByVal hInternetSession As Long, _    
                                          ByVal lpszUrl As String, _    
                                          ByVal lpszHeaders As String, _    
                                          ByVal dwHeadersLength As Long, _    
                                          ByVal dwFlags As Long, _    
                                          ByVal dwContext As Long) As Long   

Private Function GetHtmlData(lngSize As Long, strURL As String) As String   

    Dim bData() As Byte, lngReturn As Long, hFile As Long, hOpen As Long, strTmp As String   

    ReDim bData(1 To lngSize): lngReturn = 0    

    hOpen = InternetOpen("Firendless", 1, vbNullString, vbNullString, 0)    
    hFile = InternetOpenUrl(hOpen, strURL, vbNullString, ByVal 0&, &H80000000, ByVal 0&)    

    Call InternetReadFile(hFile, bData(1), lngSize, lngReturn)    

    If lngReturn = 0 Then   
        GetHtmlData = "": Call InternetCloseHandle(hOpen): Exit Function   
    End If   

    strTmp = LeftB$(bData, lngReturn)    

    Debug.Print "成功获取数据大小：" & lngSize    
    GetHtmlData = StrConv(strTmp, vbUnicode)    

    Call InternetCloseHandle(hOpen)    

End Function   

Private Sub cmdCommand1_Click()    
    MsgBox GetHtmlData(500, "http://www.promiseforever.com")    
End Sub
```

