# 修复CHROME

今天发现chrome又有了新的测试版本，果断更新了，然后果断悲剧了。 

之后下载了V19的官方下载器，不能覆盖安装，又使用离线安装包，依然不能安装。 猜测是注册表信息阻碍重新安装，于是搜索google支持中心，找到了解决方法。

<!-- more -->

原文链接: https://support.google.com/chrome/bin/answer.py?hl=zh-Hans&answer=111899

首先将下面的注册表信息保存为reg文件导入你的系统.

```reg
Windows Registry Editor Version 5.00

; WARNING, this file will remove Google Chrome registry entries  
; from your Windows Registry. Consider backing up your registry before
; using this file: http://support.microsoft.com/kb/322756

; To run this file, save it as 'remove.reg' on your desktop and double-click it.

[-HKEY_LOCAL_MACHINE\SOFTWARE\Classes\ChromeHTML] 
[-HKEY_LOCAL_MACHINE\SOFTWARE\Clients\StartMenuInternet\chrome.exe] 
[HKEY_LOCAL_MACHINE\SOFTWARE\RegisteredApplications]
"Chrome"=-

[-HKEY_CURRENT_USER\SOFTWARE\Classes\ChromeHTML] 
[-HKEY_CURRENT_USER\SOFTWARE\Clients\StartMenuInternet\chrome.exe] 
[HKEY_CURRENT_USER\SOFTWARE\RegisteredApplications]
"Chrome"=-

[-HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Uninstall\Chrome]
[-HKEY_CURRENT_USER\Software\Google\Update\Clients\{8A69D345-D564-463c-AFF1-A69D9E530F96}]
[-HKEY_CURRENT_USER\Software\Google\Update\ClientState\{8A69D345-D564-463c-AFF1-A69D9E530F96}]

[-HKEY_CURRENT_USER\Software\Google\Update\Clients\{00058422-BABE-4310-9B8B-B8DEB5D0B68A}]
[-HKEY_CURRENT_USER\Software\Google\Update\ClientState\{00058422-BABE-4310-9B8B-B8DEB5D0B68A}]

[-HKEY_LOCAL_MACHINE\SOFTWARE\Google\Update\ClientStateMedium\{8A69D345-D564-463c-AFF1-A69D9E530F96}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Google\Update\Clients\{8A69D345-D564-463c-AFF1-A69D9E530F96}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\Google\Update\ClientState\{8A69D345-D564-463c-AFF1-A69D9E530F96}]

[-HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Google\Update\Clients\{8A69D345-D564-463c-AFF1-A69D9E530F96}]
```

然后，根据系统的不同，在运行中输入不同的路径，定位到不同的目录。

- Windows XP：%USERPROFILE%\Local Settings\Application Data\Google
- Windows Vista：%LOCALAPPDATA%\Google

找到chrome文件夹，删除之。 接着重新进行安装就大功告成了。

