# [WIN32]找不到附属汇编 Microsoft.VC90.CRT

错误关键词:
找不到附属汇编 Microsoft.VC90.CR,参照的汇编没有安装在系统上,Generate Activation Context,Resolve Partial Assembly 为 Microsoft.VC90.CRT 失败...

下午配置VPS的web server.看到报错.第一直觉,尼玛,这个VPS系统没安装新的VC库...

于是下载了最新的VC 2010 SP1...安装后..继续报错..
接着下载了比较新的VC 2010 ...依旧报错..
然后下载了VC 2005...报错...

这个时候,看了一下版本..VC9?..百度得到.2008的库正式咱要的..

Microsoft Visual C++ 2008 Redistributable 
http://download.microsoft.com/download/9/7/7/977B481A-7BA6-4E30-AC40-ED51EB2028F2/vcredist_x86.exe

下载,安装..WEB SERVER亮了- -！


