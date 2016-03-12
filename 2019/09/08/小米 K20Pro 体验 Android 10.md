# 小米 K20Pro 体验 Android 10

前几天，谷歌[官方更新](https://www.android.com/android-10/)了 Android 10，对于需要在新系统上做兼容性测试（理由不错吧）的同学，入手一款 Android 10 设备的必要性不言而喻。

本想着官方会提供完善的刷机说明，没想到适配列表还是寥寥，故将刷机过程进行整理，希望能够帮助到有需要的同学。

<!-- more -->

## 写在前面

![醒目的版本号](https://attachment.soulteary.com/2019/09/08/android10.jpg)

由于各种各样的原因，目前购买的国行手机，还支持刷写 ROM 的型号/厂商其实已经不多了，小米就是其中之一。

8月29日，在刷 [小米海外社区](https://xiaomi.eu/community) 的时候，我惊喜的发现，**小米9** 和 **小米K20Pro** 支持刷写 Android 10。

![小米各型号支持刷写的系统版本](https://attachment.soulteary.com/2019/09/08/rom-list.png)

于是果断飞奔至三里屯小米之家，购入了一台 K20Pro，开始了刷机之旅。

## 解锁刷机锁

想刷小米手机，首先需要解除刷机锁。

访问小米解锁网站，获取解锁工具包，并提交解锁申请：

```TeXT
unlock.miui.com
```

官方会根据你的账号在网时长、解锁请求数量、官方策略等控制你的解锁等待时间，一般会有“立即”、“一天内”、“一周内”、“一月内”几个可能结果。

幸运的是，我当下就解锁成功了，并未进行漫长的等待。

## 准备刷机系统

想要刷机需要几个必须条件：

- 驱动就绪的 PC 操作系统：Windows/Mac/Linux
- 支持刷机的手机型号
- 手机 ROM 包

如果你已经解锁成功，理论来说你已经将手机驱动安装完毕。但是想要完成刷机，还需要一个很重要的工具：`ADB`。

根据你的操作系统，获取不同版本的 ADB 工具，等待后面使用。

- [ platform-tools-latest-windows.zip ](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
- [ platform-tools-latest-darwin.zip ](https://dl.google.com/android/repository/platform-tools-latest-darwin.zip)
- [ platform-tools-latest-linux.zip ](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)

## 获取 TWRP Recovery

官方默认的“恢复系统”不支持刷入自定义的 ROM，所以我们需要刷入支持刷写自定义 ROM 的“恢复系统”（下简写为 Rec）。

![官方阉割后的 REC 系统](https://attachment.soulteary.com/2019/09/08/origin-rec.jpg)

从[AndroidFileHost](https://androidfilehost.com/?w=files&flid=50678)可以获得小米设备的 Rec，但是文件命名都很奇怪，多数并不是我们能够一目了然的设备名称，而是一些代称，加之列表也很长，不利于查找。

文章开头的 ROM 列表图中，我们可以看到各种机型 ROM 的代号，比如 K20 Pro 的代号是 **raphael**，在页面中进行搜索，可以找到设备的 ROM 文件：`TWRP_raphael_K20PRO.zip`。

在[下载](https://androidfilehost.com/?w=files&flid=50678) REC 之后，我们需要将它刷入手机。

## 刷入 Recovery

![切换手机到 Fastboot 模式](https://attachment.soulteary.com/2019/09/08/fastboot.jpg)

将手机关机，按住手机的下音量键和电源键，等待手机进入 fastboot 模式，然后执行下面两条命令：

```bash
fastboot flash recovery twrp.img
fastboot boot twrp.img
```

不出意外，你将看到类似下面的输出：

```bash
fastboot flash recovery twrp.img

Sending 'recovery' (65536 KB)                      OKAY [  1.593s]
Writing 'recovery'                                 OKAY [  0.330s]
Finished. Total time: 1.939s
```

以及下面的日志输出：

```bash
fastboot boot twrp.img
Downloading 'boot.img'                             OKAY [  1.579s]
booting                                            OKAY [  0.141s]
Finished. Total time: 1.767s
```

此刻手机应该是开始重启，并进入了 REC 系统，界面都是中文，就不进行赘述了。

## 获取手机 ROM

在 [MIUI ROM Releases](https://xiaomi.eu/community/forums/miui-rom-releases.103/) 板块中，我们可以找到能够刷机的系统版本，比如[这个帖子](https://xiaomi.eu/community/threads/9-8-22.51929/)。帖子会指引我们去[下载文件](https://sourceforge.net/projects/xiaomi-eu-multilang-miui-roms/files/xiaomi.eu/MIUI-WEEKLY-RELEASES/9.8.22/)，在经历一番查找后，不难找到我们想要下载的[系统 ROM](https://sourceforge.net/projects/xiaomi-eu-multilang-miui-roms/files/xiaomi.eu/MIUI-WEEKLY-RELEASES/9.8.22/xiaomi.eu_multi_HMK20ProMI9TPro_9.8.22_v10-10.zip/download)，耐心等待系统下载完毕，就可以开始刷机了。

## 刷入手机 ROM

确保手机处于 REC 状态，将手机连接至 PC 电脑，如果驱动正常，你将可以在电脑的文件管理工具中看到手机的磁盘。

将下载好的手机 ROM 文件放入磁盘中，在 REC 中选择刷机，耐心等待手机刷机完成。

![开始刷机，耐心等待完成](https://attachment.soulteary.com/2019/09/08/flash.jpg)

重启手机，Android 10就到手啦。

## 效果令人惊讶的 Lens

![惊为天人的 Lens](https://attachment.soulteary.com/2019/09/08/lens.jpg)

刷机之后，更新完系统。

第一件事就是打开 Lens ，果然如官方演示，OCR 识别能力之强，令人发指。

## 最后

![拍照测试](https://attachment.soulteary.com/2019/09/08/photo-test.jpg)

![拍照测试](https://attachment.soulteary.com/2019/09/08/photo-test2.jpg)

拿到手机后，顺手拍了两张，“AI 加持”下感觉还不错，丝毫不逊妹纸的 P30Pro ，“大魔王”型号物超所值。

考虑网站性能，图已经压缩，可以放心打开浏览。（原图10M+，压缩后 170kb）。

—EOF

