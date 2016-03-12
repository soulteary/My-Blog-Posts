# [vb]关闭程序前提示

程序关闭之前的询问对话框

```vb
Option Explicit

Private Sub Form_Unload(Cancel As Integer)

 Dim Result As VbMsgBoxResult

 Result = MsgBox("是否退出?", vbQuestion + vbYesNo + vbDefaultButton1, "提示")

 If Result = vbNo Then Cancel = 1

End Sub
```

