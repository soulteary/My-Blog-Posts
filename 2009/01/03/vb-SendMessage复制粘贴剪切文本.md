# [vb]SendMessage复制粘贴剪切文本

SendMessage功能强大，下面示范一下复制剪切粘贴文本的过程。

```vb
Option Explicit   
  
Private Declare Function SendMessage _   
                Lib "user32" _   
                Alias "SendMessageA" (ByVal hwnd As Long, _   
                                      ByVal wMsg As Long, _   
                                      ByVal wParam As Long, _   
                                      lParam As Any) As Long  
  
Private Const WM_COPY = &H301   
Private Const WM_CUT = &H300   
Private Const WM_PASTE = &H302   
  
Private Sub cmdCopy_Click()   
  
With txtScore   
    .SelStart = 0   
    .SelLength = Len(txtScore)   
End With  
       
    SendMessage txtScore.hwnd, WM_COPY, 0, ByVal 0   
End Sub  
  
Private Sub cmdCut_Click()   
  
With txtScore   
    .SelStart = 0   
    .SelLength = Len(txtScore)   
End With  
  
    SendMessage txtScore.hwnd, WM_CUT, 0, 0   
End Sub  
  
Private Sub cmdPaste_Click()   
       
    SendMessage txtTarget.hwnd, WM_PASTE, 0, 0   
  
End Sub  
  
Private Sub cmdQuit_Click()   
       
    Unload Me  
  
End Sub  
  
Private Sub Form_Load()   
       
    txtScore.Text = "要被复制的字符Www.PromiseForever.Com"  
  
End Sub 
```


