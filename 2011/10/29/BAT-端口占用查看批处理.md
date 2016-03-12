# [BAT]端口占用查看批处理

之前有发过一篇用cmd找出占用端口程序并结束占用端口程序的日志:[CMD便捷找到占用端口的程序](http://promiseforever.com/2011/10/11/win32cmd%e4%be%bf%e6%8d%b7%e6%89%be%e5%88%b0%e5%8d%a0%e7%94%a8%e7%ab%af%e5%8f%a3%e7%9a%84%e7%a8%8b%e5%ba%8f.html)。 


但是，指令毕竟不如直接交互的程序来的简易。转了一个bat保存下来。
流程挺简单，就不多解释了。可以考虑再写一个vbe版本的玩。

[![2011-10-29_024054](https://attachment.soulteary.com/2011/10/29/2011-10-29_024054.gif "2011-10-29_024054")](https://attachment.soulteary.com/2011/10/29/2011-10-29_024054.gif)

<!-- more -->

```dos
@ECHO OFF && CLS
TITLE Port Tools. Ver 1.0


:InputPortA
CLS
SET "port="
SET /P port=输入想要查看的端口号(0-65535):
IF NOT DEFINED port GOTO InputPortA
FOR /F "tokens=* delims=0123456789" %%a IN ("%port%") DO IF "%%a" NEQ "" GOTO InputPortA
IF %port% GTR 65535 GOTO InputPortA
CLS
ECHO 当前活动连接
ECHO   协议   本地地址:端口          外部地址:端口          状态            PID
netstat -ano | findstr "%port%"
GOTO Info


:Info
SET /P msgstr=继续查看端口使用情况/结束占用端口的进程(v/c):
IF /I "%msgstr%"=="v" GOTO InputPortA
IF /I "%msgstr%"=="c" (GOTO InputPID) ELSE (GOTO Info)


:InputPID
SET "pid="
SET /P pid=输入想要结束的PID(0-65535):
IF NOT DEFINED pid GOTO InputPID
FOR /F "tokens=* delims=0123456789" %%a IN ("%pid%") DO IF "%%a" NEQ "" GOTO InputPID
IF %port% GTR 65535 GOTO InputPID
CLS
ECHO 进程名称                       PID 进程名称          会话号码     内存使用
ECHO ========================= ======== ================ =========== ============
tasklist | findstr "%pid%"
SET "cho="
SET /P cho=是否结束该进程(y/n):
IF /I "%cho%"=="y" GOTO CloseUse
IF /I "%cho%"=="n" (GOTO InputPortA) ELSE (GOTO InputPID)


:CloseUse
taskkill /PID "%pid%" /T
GOTO InputPortA
```

[download id="84"] 

再贴一个批处理中的换行处理脚本。

引用：[出处](http://promiseforever.com/redirect?url=http://www.bathome.net/thread-6692-1-1.html&key=5dafd960bc8947064da7864029e2300e) 

[download id="85"]

