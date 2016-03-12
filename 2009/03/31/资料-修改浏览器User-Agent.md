# [资料]修改浏览器User-Agent

IE 修改IE的UserAgent需要编辑注册表。

```reg
"HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionInternet Settings5.0User AgentPost Platform"
"HKEY_LOCAL_MACHINESOFTWAREMicrosoftWindowsCurrentVersionInternet SettingsUser AgentPost Platform"
"HKEY_LOCAL_MACHINESOFTWAREMicrosoftWindowsCurrentVersionInternet Settings5.0User AgentPost Platform"
```

如要修改IE的UserAgent为FireFox的，可以这么做： UserAgent的默认值改为"Firefox"，同时在Post Platform下面新建字符串值"Firefox"=""，注意修改后需重启IE。 FireFox

```text
1.在地址栏输入“about:config”，按下回车进入设置菜单。
2.找到“general.useragent.override”，如果没有这一项，则点右键“新建”->“字符串”，输入这个字符串。
3.将其值设为自己想要的UserAgent。
```

Opera 方法1：

```
1.工具栏“Tools”->“Preferences”->“Content”->“Advenced”，点击“Manage Site Preferences”按钮。

2.点击“Add”按钮，在弹出的窗口中“Site”填入“*”，“Network”选项卡中选择浏览器ID。各选项如下：
0 Default
1 Opera
2 Mozilla, Opera detectable
3 Internet Explorer, Opera detectable
4 Mozilla, Opera hidden
5 Internet Explorer, Opera hidden
```

方法2：

```text
1.在地址栏输入“opera:config”，回车打开。
2.找到“User Agent”点开，里面的“Spoof UserAgent ID”设置想要的值，范围1-5，具体对应的ID同上。
```

Maxthon

```text
工具栏“工具”->“遨游设置中心”->“高级选项”，勾选“自定义 UserAgent 字符串”，下面写上自己的UserAgent记可。保存设置后重启Maxthon生效。
```

Chrome

```text
方法一：启动时加上参数：--user-agent="你自己的UserAgent"

方法二：修改chrome.dll。把里面疑似UserAgent的字符串改为自己的。
```

Safari

```text
1.菜单栏“Edit”->“Preferences”->“Advanced”，勾选“Show Develop menu in menu bar”。
2.菜单栏会多出一项“Develop”，通过里面的“User Agent”子菜单即可设置自己的UserAgent。
```

iPhone

```text
把/System/Library/Frameworks/WebKit.framework/WebKit 文件中的Mozilla/5.0替换成其他UA，字符数不要超过“Mozilla/5.0”的长度。
```

