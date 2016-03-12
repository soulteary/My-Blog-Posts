# VB中让Picturebox底色透明

_想要让picturebox的底色变成透明的么？仔细看下面的说明吧。_ 

## 说明

想要获得上面说的效果,你必须先在组件面板中添加“Microsoft Forms2.0 Object Library”，然后用新添加上的PictureBox控件。

此控件和默认的PictureBox没有什么区别，只是属性中多出了BackStyle属性，使用这个属性，将控件设为背景透明0-fmBackStyleTransparent，这样控件的背景就能和窗体的背景保持一致了。

## 参考资料

Microsoft Forms 2.0 Object Library实际上是IE提供的控件，它主要是弥补HTML提供的控件的不足。

同VB5的控件相比，基本功能相同，但多几个属性(有的控件少几个属性)，这你可以在属性窗口中查看。

由于功能相同，而一般VB的开发者不愿意在发行软件时再带上FM20.DLL，所以一般VB的设计者不使用。

但如果你使用VB6设计DHTML程序，你只能使用这些控件而不能使用VB自身的控件了。Ms Forms 2.0 Object Library的帮助参见“Program FilesCommon FilesMicrosoft SharedVBAFm20.hlp”。

