# vold进程资料

一、进程启动和配置文件的分析
 
  vold的全称是volume daemon。实际上是负责完成系统的CDROM, USB大容量存储，MMC卡等扩展存储的
  挂载任务自动完成的守护进程。它提供的主要特点是支持这些存储外设的热插拔。在Android上的这个 
  vold系统和GNU/Linux的之间存在很大的差异，这里主要是分析Android上的vold系统的处理过程。
  自Android 2.2开始，vold又做了大改动，升级为vold 2.0，之前的配置文件是
      system/etc/vold.conf，vold 2.0变为system/etc/vold.fstab。

<!-- more -->

1、启动vold
  
    在init.rc中启动VOLD这个守护线程和创建socket的命令如下：
    service vold /system/bin/vold
        socket vold stream 0660 root mount
        ioprio be 2
     
2、配置vold.fstab
  
    vold.fstab文件的格式是：
    Format: dev_mount <label> <mount_point> <part> <sysfs_path1...>
    label:    -Label for the volume
    mount_point  -Where the volume will be mounted
    part     -Partition #(1 based), or 'auto' for first usable partition.
    <sysfs_path> -List of sysfs paths to source devices
    
例如：
    dev_mount sdcard /mnt/sdcard 1 /devices/platform/mxsdhci.0/mmc_host/mmc0
    
自Android 2.2后，SD mount的位置变为/mnt/sdcard。
      
二、控制流程分析
 
Vold关于SD card settings的代码位于：
packages/apps/Settings/src/com/android/settings/deviceinfo/Memory.java
Vold上层MountService的代码位于：
frameworks/base/services/java/com/android/server/MountService.java
Vold底层处理的代码位于：
system/vold/
    
  1、Vold设计架构
    
```
Setting
 |
MountService
 |
CommandListener
 |
VolumeManager  - NetlinkManager
 |
Volume  -  DirectVolume
 |
SD/USB device
```
   
MountService会接收来之Setting的变化，及来自底层VolumeManager的信息，并对之分析判，然后
通过doMountVolume命令到底层。
Vold初始化时，会创建class NetlinkManager和VolumeManager，class NetlinkManager接收
来自底层的信息，然后传交给VolumeManager处理；
重要类class VolumeManager 仅有一个实例，它主要负责vold的管理操作，管理多个sd卡，usb各种
操作；重要类class Volume 可有多个实例， 挂载多少个sd卡、usb，它就有多少个。重要类class 
DirectVolume 封装了很多的方法和属性；重要类class CommandListener主要收到上层
MountService通过doMountVolume发来的命令，分析后，转交给VolumeManager处理；
VolumeManager处理信息后，或报告给上层MountService,或交给volume执行具体操作（挂载
SD,USB）.      
    
2、Vold代码实现过程大致分为三步：
  
1）.创建链接：
 在vold作为一个守护进程，一方面接受驱动的信息，并把信息传给应用层；另一方面接受上层的命令并
 完成相应操作。
 所以这里的链接一共有两条：
 （1）vold socket: 负责vold与应用层的信息传递；
 （2）访问udev的socket: 负责vold与底层的信息传递；
 这两个链接都是在进程的一开始完成创建的。
 
2）.引导：
 这里主要是在vold启动时，对现有外设存储设备的处理。首先，要加载并解析vold.fstab，
 并检查挂载点是否已经被挂载（注：这里检查挂载点的用意不是很清楚！）； 其次，执行MMC卡挂
 载； 最后，处理USB大容量存储。
 
3）.事件处理：
 这里通过对两个链接的监听，完成对动态事件的处理，以及对上层应用操作的响应。

