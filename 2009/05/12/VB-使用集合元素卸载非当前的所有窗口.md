# [VB]使用集合元素卸载非当前的所有窗口

[VB]使用集合元素卸载非当前的所有窗口 或许你觉得卸载窗口很简单，但是窗口如果很多呢。 看看我的代码吧。

```vb
Private Sub cmdShow_Click()
'显示所有的窗口
Form2.Show
Form3.Show
End Sub

Private Sub cmdUnload_Click()
'卸载非当前窗口
Dim frm As Form

For Each frm In Forms

If frm.Name <> "Form1" Then Unload frm

Next

End Sub
```

