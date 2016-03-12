# [随笔]HP TX2121 BIOS修复

[![tx2121](https://attachment.soulteary.com/2011/02/17/tx2121.jpg "tx2121")](https://attachment.soulteary.com/2011/02/17/tx2121.jpg) 

今天睡醒吃了口面就跑出去了，昨天没有准备，没有完成。

今天去了后，试验hp bios recovery mode，最后发现可以进入，很犀利。 

之前还电话咨询了一下hp客服，客服淡定的回答我，这个不属于技术支持，叫我去售后- -。说盲刷BIOS怎么样等等.. 

无奈只能自己动手了。  

于是，google了一下“compaq bios recovery mode”。 其中有一篇 “[Recovery Bios on HP Compaq CQ40 and HP Pavilion dv4-2160, cq5107c, dv4-2164](http://www.laptopndriver.com/hp/recovery-bios-on-hp-compaq-cq40-and-hp-pavilion-dv4-2160-cq5107c-dv4-2164.html)”感觉比较靠谱，按着上面的提示。 

1. 准备一份新的BIOS文件[我准备的是加入SLIC2.1表的修改BIOS...] 
2. 修改短文件名,符合DOS8文件名长度规则,并准备FAT格式启动U盘[相关工具,稍后上传] 
3. 给笔记本断开电源,关闭计算机,移除电池. 
4. 我是将USB插入笔记本右侧的USB口,长按WIN+B,插入电源线,按着POWER键启动笔记本。 
5. 笔记本规则的2秒一声报警,USB数据灯狂闪10多秒,机器自动关闭. 
6. 启动笔记本,发现可以启动了,而且支持SATA了. 记录一下,希望这个帖子可以帮助到HP DV系列的朋友,尤其是刷BIOS,或者GHOST或者因为使用WIN7激活出现问题,黑屏,不能启动,只是显示LOGO等情况的朋友。


首先上传USB格式化工具.不会使用的话,可以本帖留言。

[HPUSB](https://attachment.soulteary.com/2011/02/HPUSB.rar) 

然后是修改过的BIOS,TX2121测试可以正常使用。 

[306DF06](https://attachment.soulteary.com/2011/02/306DF06.rar) 

后面带上今天的流水账，柳巷云路街【棉花巷】哪家咖啡店还是没记住名字..不是玛雅.感觉东西不好喝...下次定然不去了.. 

很羡慕高原，加油啊，希望你们完美。

