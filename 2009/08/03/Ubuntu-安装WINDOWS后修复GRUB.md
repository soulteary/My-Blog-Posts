# [Ubuntu]安装WINDOWS后修复GRUB

昨天睡觉的时候重新安装WIN2K3,然后瞌睡直接睡了,今天下午回家,发下我的GRUB消失了... 

无奈,肯定是昨天安装完毕,没有修复...于是把9.04的LiveCD放进光驱... 

接着进入Live System,打开终端。 

输入 `sudo -i`，获得root权限后进入Grub：`grub`

叫程序自动查找原始的分区所在。

`find /boot/grub/stage1`

看到回显后开始修复

```bash
root (hdS,Y)
setup (hd0)
quit
```



