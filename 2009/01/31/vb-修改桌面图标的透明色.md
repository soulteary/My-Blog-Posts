# [vb]修改桌面图标的透明色

如何使桌面图标透明

```vb
Option Explicit

Private Declare Function FindWindow _
Lib "user32" _
Alias "FindWindowA" (ByVal lpClassName As String, _
 ByVal lpWindowName As String) As Long

Private Declare Function FindWindowEx _
Lib "user32" _
Alias "FindWindowExA" (ByVal hWnd1 As Long, _
 ByVal hWnd2 As Long, _
 ByVal lpsz1 As String, _
 ByVal lpsz2 As String) As Long

Private Declare Function InvalidateRectBynum _
Lib "user32" _
Alias "InvalidateRect" (ByVal hwnd As Long, _
ByVal lpRect As Long, _
ByVal bErase As Long) As Long

Private Declare Function SendMessage _
Lib "user32" _
Alias "SendMessageA" (ByVal hwnd As Long, _
ByVal wMsg As Long, _
ByVal wParam As Long, _
lParam As Any) As Long
Dim Parent&amp;, Child&amp;, CChild&amp;

Private Sub Command1_Click()

Parent = FindWindow(vbNullString, "Program Manager")

Child = FindWindowEx(Parent, 0, "SHELLDLL_DefView", vbNullString)

CChild = FindWindowEx(Child, 0, "SysListView32", vbNullString)

SendMessage CChild, (&amp;H1000 + 38), 0, ByVal RGB(255, 255, 255)

InvalidateRectBynum CChild, 0, True
 
End Sub

Private Sub Command2_Click()

Dialog1.ShowColor

Parent = FindWindow(vbNullString, "Program Manager")
Child = FindWindowEx(Parent, 0, "SHELLDLL_DefView", vbNullString)
 
CChild = FindWindowEx(Child, 0, "SysListView32", vbNullString)
 
SendMessage CChild, (&amp;H1000 + 38), 0, ByVal Dialog1.Color
InvalidateRectBynum CChild, 0, True
End Sub
```
 

如果不成功，参考下面的方法。

电脑-属性-高级-性能项- 设置-自定义-在桌面上为图标标签使用阴影-打勾

如果不行，再在桌面空白处右击-排列图标-在桌面上锁定web项目-把勾去掉

可以尝试以下4种方法：

1. 右击“我的电脑”，依次单击“属性/高级/性能设置”在“视觉效果”页中将“在桌面上为图标标签使用阴影”选中，单击确定即可。
2. 右键桌面空白处右击，在“排列图标”里去掉“锁定桌面的web项目”
3. 有时会出现上述设置也不能解决问题，我们就可以通过新建一个用户的办法解决，但桌面图标、快速启动栏以及环境变量等等设置会恢复为默认状态，需要重新设置。(一般不用这项)
4. 另一种方法也可轻松解决问题：右击桌面空白处，依次单击“属性/桌面/自定义桌面/web”选项，将“网页”栏中的“当前主页”以及“http//......”等所有各项前面的勾全部去掉（“http//……”为从Internet添加网页或图片的地址，一般不需要，可将它们全部删除），并将下面“锁定桌面项目”前面的勾也去掉，单击确定完成设置，就又能看到可爱的桌面图标了。

另外有一种情况就是安装了某种程序之后(比如系统提示:是否将该Active Desktop项添加到您的桌面上)，桌面文字变的不透明。在“运行”中输入“gpedit.msc”，打开组策略；在“用户配置→管理模板→桌面→Active Desktop”中，点 启用Active Desktop(活动桌面)然后点击“属性”选定“已禁用”，点禁用Active Desktop (活动桌面)“属性”选定“已启用”；之后打开控制面板，在经典视图中打开系统，在“性能→高级选项→性能→视觉效果→使桌面文字透明”（等价于在之后执行第1种方法）。

