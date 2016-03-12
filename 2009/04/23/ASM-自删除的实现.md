# [ASM]自删除的实现

感谢看雪的 StarsunYzL 分享

```c
#include <windows.h>
#include <tchar.h>HMODULE hDll;

extern "C" __declspec(dllexport) void DeleteMe()
{
//在这里干其它想干的事，如删除其它exe文件

//下面代码实现DLL自删除
TCHAR* szDll = (TCHAR*)VirtualAlloc(NULL, MAX_PATH, MEM_COMMIT, PAGE_READWRITE);
GetModuleFileName(hDll, szDll, MAX_PATH);

__asm
{
push 0 ;参数1
push szDll ;参数2
push ExitProcess
push hDll ;参数3
push DeleteFile
push FreeLibrary
ret
}
}

BOOL APIENTRY DllMain(HMODULE hModule,
DWORD ul_reason_for_call,
LPVOID lpReserved
)
{
switch (ul_reason_for_call)
{
case DLL_PROCESS_ATTACH:
hDll = hModule;
break;
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}</tchar.h> </windows.h>
```


将代码编译为test.dll，然后rundll32?test.dll,DeleteMe运行，test.dll就自己删除了

那段__asm代码就是精华所在了

执行ret后，就返回到FreeLibrary处去执行，这时候ESP+4就是FreeLibrary的参数，也就是相当于调用了FreeLibrary(参数3)，而参数3是DLL自身的模块句柄，所以相当于DLL自己把自己从rundll32.exe里给卸载了；

FreeLibrary执行完后会将参数3出栈，并返回到DeleteFile处去执行，这时相当于调用了DeleteFile(参数2)，参数2就是DLL文件自身的路径啦，这个路径必须存放在用VirtualAlloc在rundll32.exe里分配的内存，因为这时DLL已经被卸载了；

同理最后调用的是ExitProcess(参数1)，防止rundll32继续运行下去出错。

这样DLL就可以实现自删除啦，这有啥用捏？借助这个DLL，可以实现EXE的自删除，例如把这个DLL放到一个EXE里，当EXE需要自删除的时候，先释放出DLL，然后把EXE自身的路径告诉DLL，最后CreateProcess??rundll32?xxx.dll,DeleteMe，DeleteMe里先删除EXE，再自删除，就可以实现EXE的自删除啦


