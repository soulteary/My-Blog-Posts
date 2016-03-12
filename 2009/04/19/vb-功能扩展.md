# [vb]功能扩展

我们在设计一个VB窗体时， 常常放上许多控件， 为了使这些控件看上去整整齐齐，

我们不得不设置一大堆Left、 Top、 Height、 Width属性， 

您是否已经感到厌烦并想转向Powerbuilder或 Delphi等别急，让我们现在就来扩展一下VB的功能。

VB提供了一个功能:Add-Ins，利用这一功能我们就可以把自己的程序加到VB的系统菜单Add-Ins里去，作为VB的扩展功能。

我们设计的程序将具有以下功能：选取窗体上某些控件后，通过菜单选取，使它们大小相同、间距相同、边缘对齐等等。

有了这些功能,我们设计界面时就能节省大量时间,大大提高工作效率。<!--more-->

限于篇幅,这里只介绍其中一个功能:使所选取控件从左到右大小相同。

理解了这段程序,其它功能就很容易实现了。

首先建一个新项目:alignment.mark,不需要任何窗体,在Tools菜单里选Project Options，将 Project Name设为Exam,将Start Mode设为Object Application后退出。然后在菜单Ins ert里选取ClassModule,建立一个新类,属性设置如下:

```vb

Name="HSizeAlign";Creatable=False;Public=True 输入以下程序: 

Public VBInstance As Object '当前所运行的VB

Private TheseControls As Object

Private Control As Onject '控件对象变量

Private AllHeight As Long

Private AllWidth As Long

Private MinLeft As Long '标记最左边界值

Public Sub AfterClick()

MinLeft=99999 '设一极大初值

Set ThereControls = VBInstance.ActiveProject.Ac - tiveForm.SelectedControlTemplates

For Each Control In TheseControls

If Control.Properties("Left")<MinLeft Then

AllHeight=Control.Properties("Height")

AllWidth=Control.Properties("Width")

MinLeft=Control.Properties("Left")

End If

Next

For Each Control In TheseControls

Control.Properties("Height")=AllHeight

Control.Properties("Width")=AllWidth

Next

End Sub
```

再定义一个新类,属性设置如下:

```vb

Name="Alignment";Creatable=True;Public=True

Dim ThisInstance As Object

Dim HSizeAlignMenu As Object

Dim HSizeAlignHandler As New HSizeAlign

Dim HSizeConnectCookie As Long

Sub ConnectAddIn(VBInstance As Object)

'加入菜单项,进行连接

Set ThisInstance=VBInstance

Set HSizeAlignMenu=ThisInstance.AddinMenu.

MenuItems.Add("HSize Alignment")

Set HSizeAlignHandler.VBInstance=ThisInstance

HSizeConnectCookie=HSizeAlignMenu.ConnectEvents

(HSizeAlignHandler)

End Sub

Sub DisconnectAddIn(Mode As Integer)

'解除连接,删除菜单项

HSizeAlignMenu.DisconnectEvents HSizeConnect-Cookie

ThisInstance.AddinMenu.MenuItems.Remove HSizeAlignMenu

End Sub
```

再加入一个Module,输入以下程序:

```vb
Declare Function WritePrivateProfileString Lib

"KERNEL"(ByVal AppName$,ByVal KeyName$,ByVal keydefault$,ByVal FileName$)

Declare Function GetPrivateProfileString Lib

"KERNEL"(ByVal AppName$,ByVal KeyName$,ByVal keydefault$,ByVal ReturnString$,By

Val NumBytes As Integer,ByVal FileName$)

'以上说明可用API Text Viewer拷贝

Sub Main()

Dim ReturnString As String

Section$="Add-Ins16"

ReturnString=String$(255,Chr$(0))

ErrCode=GetPrivateProfileString(Section$,

"Exam.Alignment","NotFound",ReturnString,Len(ReturnString)+1,"VB.INI")

If Left(ReturnString,InStr(ReturnString,Chr(0))-1)="NotFound"Then

ErrCode=WritePrivateProfileString%(Section$,"Exam.Alignment","0","VB.INI")

End If

End Sub
```

'Exam.Alignment里,Exam"为项目名,Alignment"为与Add-In菜单连接的类名。

以上程序编译运行后,在VB菜单Add-Ins里选取Add-In Manager,将弹出一对话框,选取Exam. Alignment后退出,Add-Ins菜单里就多了一项HSize Alignment;使用时先选取所需排列控件 ,然后选此菜单项即可。

