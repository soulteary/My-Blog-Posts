# [vb]使用CMD强制关闭进程

使用CMD强制关闭进程

<!-- more -->

步骤：

一、打开“运行”，输入CMD,打开一个模拟的DOS窗口。
二、输入`ntsd -c q -p Pid`

其中Pid所要关闭的程序的进程数，WINDOWSXP下查看方法：打开任务管理器->查看-->选择列-->勾选PID,确定。此时在进程后面就有了该进程的PID值。

在cmd窗口查看进程更加快捷,同时也可查看到在任务管理器中不显示的进程,以便有效监视系统的运行,防止病毒的运行

```text
:tasklist:显示当前系统所有进程命令
:taskkill:关闭至少一个系统进程命令

    /pid 后面是系统的进程id 如先查到notepad.exe的id是2152,则格式为 taskkill /pid 2152
         多个时格式为 taskkill /pid 2152 /pid 1284
    /im 后面是系统的进程名     如要关闭notepad.exe,格式为taskkill /im notepad.exe,指定多个       时格式为taskkill /im notepad.exe /im iexplorer.exe .如果是要关闭所有的,则使用通配符*,即Taskkill /im *.exe
    /T 后面是结合上两个命令实现,如taskkill /t /im notepad.exe或者taskkill /t /pid 2152   这个效果是提示后在使用者确定后关闭,有提示框.
    /F   后面也是结合/pid 和/im实现,如taskkill /F /im notepad.exe或者taskkill /F /pid 2152
```

同时也可以在"开始-->运行-->msconfig"中查看到运行的程序,但相对cmd稍嫌繁琐了点.

