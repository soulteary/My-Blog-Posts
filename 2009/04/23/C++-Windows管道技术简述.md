# [C++]Windows管道技术简述

Windows管道技术简述 detrox 不知你是否用过这样的程序，他们本身并没有解压缩的功能，而是调用DOS程序PKZIP完成ZIP包的解压缩。但是在程序运行时又没有DOS控制台的窗口出现而且一切本应该在DOS下显示的信息都出现在了那个安装程序的一个文本框里。这种设计既美观又可以防止少数眼疾手快的用户提前关了你的DOS窗口。 现在就来讨论一下，如何用匿名管道技术实现这个功能。 管道技术由来已久，相信不少人对DOS命令里的管道技术最为熟悉。当我们type一个文件的时候如果想让他分页现实可以输入 C:>type autoexec.bat|more 这里“|”就是管道操作符。他以type输出的信息为读取端，以more的输入端为写入端建立的管道。 Windows中使用较多的管道也是匿名管道，它通过API函数CreatePipe创建。

```c
BOOL CreatePipe(
PHANDLE hReadPipe, // 指向读端句柄的指针
PHANDLE hWritePipe, // 指向写端句柄的指针
LPSECURITY_ATTRIBUTES lpPipeAttributes, // 指向安全属性结构的指针
DWORD nSize // 管道的容量
);
```

上面几个参数中要注意hReadPipe,hWritePipe是指向句柄的指针，而不是句柄（我第一次用的时候就搞错了）。nSize一般指定为0,以便让系统自己决定管道的容量。现在来看安全属性结构，SECURITY_ATTRIBUTES。

```c
typedef struct _SECURITY_ATTRIBUTES { // sa
DWORD nLength;
LPVOID lpSecurityDescriptor;
BOOL bInheritHandle;
} SECURITY_ATTRIBUTES;
```

nLength是结构体的大小，自然是用sizeof取得了。lpSecurityDescriptor是安全描述符（一个C-Style的字符串）。bInheritHandle他指出了安全描述的对象能否被新创建的进程继承。先不要管他们的具体意义，使用的时候自然就知道了。 好，现在我们来创建一个管道

```c
HANDLE hReadPipe, hWritePipe;
SECURITY_ATTRIBUTES sa;
sa.nLength = sizeof(SECURITY_ATTRIBUTES);
sa.lpSecurityDescriptor = NULL; //使用系统默认的安全描述符
sa.bInheritHandle = TRUE; //一定要为TRUE，不然句柄不能被继承。
CreeatePipe(&amp;amp;hReadPipe,&amp;amp;hWritePipe,&amp;amp;sa,0);
```

OK,我们的管道建好了。当然这不是最终目的，我们的目的是把DOS上的一个程序输出的东西重定向到一个Windows程序的Edit控件。所以我们还需要先启动一个DOS的程序，而且还不能出现DOS控制台的窗口（不然不就露馅了吗）。我们用CreateProcess创建一个DOS程序的进程。

```c
BOOL CreateProcess(
LPCTSTR lpApplicationName, // C-style字符串:应用程序的名称
LPTSTR lpCommandLine, // C-style字符串:执行的命令
LPSECURITY_ATTRIBUTES lpProcessAttributes, // 进程安全属性
LPSECURITY_ATTRIBUTES lpThreadAttributes, // 线程安全属性
BOOL bInheritHandles, // 是否继承句柄的标志
DWORD dwCreationFlags, // 创建标志
LPVOID lpEnvironment, // C-Style字符串：环境设置
LPCTSTR lpCurrentDirectory, // C-Style字符串：执行目录
LPSTARTUPINFO lpStartupInfo, // 启动信息
LPPROCESS_INFORMATION lpProcessInformation // 进程信息
);
```

先别走，参数是多了点，不过大部分要不不用自己填要不填个NULL就行了。lpApplication随便一点就行了。lpCommandLine可是你要执行的命令一定要认真写好。来，我们瞧瞧lpProcessAttributes和lpThreadAttributes怎么设置。哎？这不就是刚才那个吗。对阿，不过可比刚才简单。由于我们只是创建一个进程，他是否能在被继承不敢兴趣所以这两个值全为NULL。bInHeritHandles也是一定要设置为TRUE的，因为我们既然要让新的进程能输出信息到调用他的进程里，就必须让新的进程继承调用进程的句柄。我们对创建的新进程也没什么别的苛求，所以dwCreationFlags就为NULL了。lpEnvironment和lpCurrentDirectory根据你自己的要求是指一下就行了，一般也是NULL。接下来的lpStartupInfo可是关键，我们要认真看一下。

```c
typedef struct _STARTUPINFO { // si
DWORD cb;
LPTSTR lpReserved;
LPTSTR lpDesktop;
LPTSTR lpTitle;
DWORD dwX;
DWORD dwY;
DWORD dwXSize;
DWORD dwYSize;
DWORD dwXCountChars;
DWORD dwYCountChars;
DWORD dwFillAttribute;
DWORD dwFlags;
WORD wShowWindow;
WORD cbReserved2;
LPBYTE lpReserved2;
HANDLE hStdInput;
HANDLE hStdOutput;
HANDLE hStdError;
} STARTUPINFO, *LPSTARTUPINFO;
```

倒！这么多参数，一个一个写肯定累死了。没错，MS早就想到会累死人。所以提供救人一命的API函数GetStartupInfo。

```c
VOID GetStartupInfo(
LPSTARTUPINFO lpStartupInfo
);
```

这个函数用来取得当前进程的StartupInfo,我们新建的进程基本根当前进程的StartupInfo差不多，就借用一下啦。然后再小小修改一下即可。 我们要改的地方有这么几个：cb,dwFlags，hStdOutput，hStdError，wShowWindow。先说cb，他指的是STARTUPINFO的大小，还是老手法sizeof。再说wShowWindow,他制定了新进程创建时窗口的现实状态，这个属性当然给为SW_HIDE了，我们不是要隐藏新建的DOS进程吗。哈哈，看到hStdOutput和hStdError，标准输出和错误输出的句柄。关键的地方来了，只要我们把这两个句柄设置为hWrite,我们的进程一旦有标准输出，就会被写入我们刚刚建立的匿名管道里，我们再用管道的hReadPipe句柄把内容读出来写入Edit控件不就达到我们的目的了吗。呵呵，说起来也真是听容易的阿。这几个关键参数完成了以后，千万别忘了dwFlags。他是用来制定STARTUPINFO里这一堆参数那个有效的。既然我们用了hStdOutput,hStdError和wShowWindow那dwFlags就给为STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES。 好了，现在回到CreateProcess的最后一个参数lpProcessInformation（累！）。呵呵，这个参数不用自己填了，他是CreateProcess返回的信息，只要给他一个PROCESS_INFORMATION结构事例的地址就行了。 大功高成了，我们管道一端连在了新进程的标准输出端了，一端可以自己用API函数ReadFile读取了。等等，不对，我们的管道还有问题。我们把hWrite给了hStdOutput和hStdError,那么在新的进程启动时就会在新进程中打开一个管道写入端，而我们在当前进程中使用了CreatePipe创建了一个管道，那么在当前进程中也有这个管道的写入端hWrite。好了，这里出现了一个有两个写入端和一个读出端的畸形管道。这样的管道肯定是有问题的。由于当前进程并不使用写端，因此我们必须关闭当前进程的写端。这样，我们的管道才算真正的建立成功了。来看看VC++写的源程序：

```c
/*
* 通过管道技术，将dir /?的帮助信息输入到MFC应用程序的一个CEdit控件中。
* VC++6.0 + WinXP 通过
*
* detrox, 2003
*/

void CPipeDlg::OnButton1()
{
SECURITY_ATTRIBUTES sa;
HANDLE hRead,hWrite;

sa.nLength = sizeof(SECURITY_ATTRIBUTES);
sa.lpSecurityDescriptor = NULL;
sa.bInheritHandle = TRUE;
if (!CreatePipe(&amp;amp;hRead,&amp;amp;hWrite,&amp;amp;sa,0)) {
MessageBox("Error On CreatePipe()");
return;
}
STARTUPINFO si;
PROCESS_INFORMATION pi;
si.cb = sizeof(STARTUPINFO);
GetStartupInfo(&amp;amp;si);
si.hStdError = hWrite;
si.hStdOutput = hWrite;
si.wShowWindow = SW_HIDE;
si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
if (!CreateProcess(NULL,"c:windowssystem32cmd.exe/c dir /?"
,NULL,NULL,TRUE,NULL,NULL,NULL,&amp;amp;si,&amp;amp;pi)) {
MessageBox("Error on CreateProcess()");
return;
}
CloseHandle(hWrite);

char buffer[4096] = {0};
DWORD bytesRead;
while (true) {
if (ReadFile(hRead,buffer,4095,&amp;amp;bytesRead,NULL) == NULL)
break;
m_Edit1 += buffer;
UpdateData(false);
Sleep(200);
}
}
```

