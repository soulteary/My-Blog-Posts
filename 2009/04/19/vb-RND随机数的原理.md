# [vb]RND随机数的原理

[vb]RND随机数的原理 Microsoft Visual Basic RND 函数中的伪随机数字生成使用线性congruential算法。 下面的伪代码文档使用的算法：

```vb
x1 = ( x0 * a + c ) MOD (2^24)
```

位置：

x  = 新的值
x0 = 以前的值 (初始值为327680)
a  = 1140671485
c  = 12820163

套用公式后,返回的整数的余数就是我们的随机数了。
表达式 x1/(2^24)将返回介于0.0到由RND函数返回的1.0之间的浮点数。
在VB内部无法实现这一运算，因为VB内部不支持无符号长整形数据。

下面的 C/C++ 代码可生成Visual Basic 生成的伪随机数字的前十个：

```c
#include "stdafx.h"   

int main(int argc, char* argv[])   
{   
unsigned long       rndVal;   

rndVal = 0x50000L;   
int i;   
float rndFloat;   

for (i=0;i<10;i++)   
 {   
 rndVal = (rndVal * 0x43fd43fdL + 0xc39ec3L) & 0xffffffL;   
 rndFloat = (float)rndVal / (float)16777216.0;   
 printf("Value is %.15f
",rndFloat);   
 }   
return 0;   
}

```

