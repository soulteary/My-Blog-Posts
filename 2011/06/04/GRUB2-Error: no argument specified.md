# [GRUB2]Error: no argument specified

首先描述一下问题情况.会出现这个问题的童鞋盆友.估计是手残的修改分区表.当然,可能你断电断的MBR,EBR GG了.
总而言之，分区表挂鸟。然后你又重新修复了分区表，然后使用新的引导盘安装GRUB2，并登陆终端，输入了update-grub...

<!-- more -->

老外朋友也有雷同的，而且有一个帅气的哥们给出了问题的解决方法，就是给set后面加一个参数，“=root”
我猜想，是因为使用LIVE CD,LIVE CD的权限没有被正确识别?[误]
所以呢,update-grub的时候,读取不到当前用户权限,或者当前权限身份不在选择列表里，于是没有输出，然后
就出现了下面这种形式的shell语句。缺少一个参数。

```text
search --no-floppy --fs-uuid --set c9101109-b746-464e-b9ec-5c62d9958c8d
```

所以，只要补全参数就好了。

```text
search --no-floppy --fs-uuid --set=root c9101109-b746-464e-b9ec-5c62d9958c8d
```

最后
@drs305 ,thanks.
原帖地址:http://ubuntuforums.org/showthread.php?t=1662142&page=2

我还是偷懒，没有给系统升级，依旧10.04.

