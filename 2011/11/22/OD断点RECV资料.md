# OD断点RECV资料

首先奉上资料链接.[原文地址](http://promiseforever.com/redirect?url=http://blog.csdn.net/liuhua1982/article/details/6319503&key=b5290b38f3d1a0eda060b90274d4d162 "OD断点RECV") 

bp recv的时候要下2个断点.至于使用WINSOCK,可以在WINSOCK2模块中下断点. 如同前人分析,

> tcp中使用的接受数据函数有2个，分别是：位于WSOCK32.dll里的recv与位于ws2_32.dll里的recv。都叫recv，以前的版本只对WSOCK32.dll里的recv进行了处理，但通如网络游戏等程序通常使用的是ws2_32.dll里的recv，所以就截取不到接受数据。

> 更无语的就是在OllyDbg里直接使用命令bp recv下断点也是下在WSOCK32.dll里的recv上，开始始终想不明白为什么，甚至怀疑socket里还有别的什么不知道的接受数据函数。所以在OllyDbg里下ws2_32.dll里recv的断点要使用bp ws2_32.recv，或者直接在模块里去找

