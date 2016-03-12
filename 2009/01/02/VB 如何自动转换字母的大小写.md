# VB 如何自动转换字母的大小写

VB中转换字母大写小写，你可以通过改变KeyAscii数值或者使用Lcase或者Ucase函数来做。

注意：使用Lcase或Ucase后，需要设置一下光标的位置。

自动转换字母的大小写[3种方法]

```vb
Private Sub txtCcase_KeyPress(KeyAscii As Integer)

If chkConvert(0).Value = 1 Then If KeyAscii > 96 And KeyAscii < 123 Then KeyAscii = (KeyAscii And 223)
If chkConvert(1).Value = 1 Then If KeyAscii > 64 And KeyAscii < 91 Then KeyAscii = (KeyAscii Or 32)

If chkConvert(2).Value = 1 Then If KeyAscii > 96 And KeyAscii < 123 Then KeyAscii = KeyAscii + 32
If chkConvert(3).Value = 1 Then If KeyAscii > 64 And KeyAscii < 91 Then KeyAscii = KeyAscii - 32

End Sub

Private Sub txtCcase_KeyUp(KeyCode As Integer, Shift As Integer)

If chkConvert(4).Value = 1 Then txtCcase = UCase$(txtCcase): txtCcase.SelStart = Len(txtCcase)
If chkConvert(5).Value = 1 Then txtCcase = LCase$(txtCcase): txtCcase.SelStart = Len(txtCcase)

End Sub
```


