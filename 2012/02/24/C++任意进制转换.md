# C++任意进制转换

```c
#include <iostream>#include <string>using namespace std;

//将一个string倒置
void reverse(string &a)
{
  char temp;
  for(int i=0; i<=(a.size()-2)/2; i++)
  {
    temp=a[i];
    a[i]=a[a.size()-i-1];
    a[a.size()-i-1]=temp;
  }
}

//默认s1为正序，s2为正序，【2－36任意进制转化】
string NtoN(string s1,long base1,long base2)
{
  string s2;
  long sum=0,yushu;
  string::iterator it;
  it=s1.begin();
  while(it!=s1.end())
  {
    sum*=base1;
    if(*it<='9' && *it>='0')
      sum+= *it-'0';
    else
      sum+= *it+10-'A';
    *it++;
  }
  cout<<"sum(decimal):"<</string></iostream>
```

