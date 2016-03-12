# HP笔记本BIOS刷新失败黑屏的解决

整理收藏夹,想起来09年帮高原童鞋换系统刷BOIS的事情了.

把这篇文章整理下吧. 有的HP笔记本带着的是Vista Home Premium 版本的Windows系统，高原的本就是其中悲剧的例子之一。

记得当时他说使用不惯，于是让我换成XP。

首先是备份原来的各种个人数据，然后收集XP版本的驱动，重新分区，以及安装新的系统。 第一步完成之后，分区结束，发现机器不能开机了，然后发现某些硬件驱动不正常，搜索一番发现是BIOS版本问题。 于是在WINXP下更新BIOS,更新之后发现笔记本直接关机了，开机后，漆黑的屏幕提示"go black after flash bios f24 on hp dv4"

解决方法也很简单:

到[美国官网](http://promiseforever.com/redirect?url=http%3A%2F%2Fwelcome.hp.com%2Fcountry%2Fus%2Fen%2Fwelcome.html&key=0e5e4bac5466a484896005244f872696)上下载最近的HP BIOS固件。

然后将下载后的文件(SPXXX.exe)用解压缩软件解压，得到若干文件，找到XXXXYYYU.FD和XXXXYYYD.FD两个文件拷贝出来。 XXXXYYYU.FD用于集成显卡(UMA)机器，XXXXYYYD.FD用于独立显卡的机器。将对应你的机器的固件复制到格式化为FAT格式的U盘上， 当时使用的是2G的KINGSTON,拷贝完成后,记得修改文件名为XXXX.BIN。 将笔记本电池取掉，拔掉AC电源，插上U盘,按下WIN+B键,插上交流电源，按下电源开关开机。然后松开WIN+B键，在听到滴滴声后，大功告成。 先是一次较长声音的“滴”，然后是一声短“滴”。这意味着找到了正确的BIOS固件。在开始刷新固件的时候，风扇似乎会狂转，机器也会一直发出“滴滴”的声音，不一会机器刷新完毕自动关机重启，记得拔掉U盘，现在你就可以干你想干的事情了。 本文适用HP DV4刷BIOS 黑屏,以及CQ45刷BIOS 黑屏,理论兼容所有使用同样BIOS的笔记本。

请在自己尝试的时候，胆大心细一些！

