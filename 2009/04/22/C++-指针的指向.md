# [C++]指针的指向

转自: http://blog.csdn.net/ecjtuync/archive/2008/11/25/3368105.aspx

```c
LPSTR, LPCSTR ,LPTSTR,LPCSTR,LPWSTR LPCWSTR

LPCSTR 32-bit指针，指向一个常量字串
LPSTR  32-bit指针，指向一个字串
LPCTSTR32-bit指针，指向一个常量字串。此字串可移植到
Unicode和DBCSLPTSTR 32-bit指针，指向一个字串。此字串可移植到Unicode和DBCS
LPWSTR 32-bit指针，指向一个unicode字符串的指针
LPCWSTR32-bit指针, 指向一个unicode字符串常量的指针

前面的L代表LONG,P就是指针的意思,C就是constant的意思, W是wide的意思，STR就是string的意思

LPSTR = char *
LPCSTR = const char *
LPWSTR = wchar_t *
LPCWSTR = const wchar_t *
LPOLESTR = OLECHAR * = BSTR = LPWSTR(Win32)
LPCOLESTR = const OLECHAR * = LPCWSTR(Win32)

LPTSTR = _TCHAR *
LPCTSTR = const _TCHAR * 
```

