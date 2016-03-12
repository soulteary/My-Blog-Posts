# PHP与ASP重定向方式比较

asp中实现重定向是用response.redirect 函数:

```asp
response.redirect "www.promiseforever.com/index.asp"
```

php中实现重定向是用header 函数:

```php
header("www.promiseforever.com/index.php");
```

两种方式的区别： asp的redirect函数可以在向客户发送头文件后起作用:

```html
<%response.redirect "www.promiseforever.com/index.asp"%>   
```

PHP代码

```php
// header函数之前不能向客户发送任何数据   

...
```

重定向a.asp文件

```asp
<%   
response.redirect "../a.asp"  
response.redirect "../b.asp"  
%>   
```

重定向b.php结论：在asp中执行redirect后不会再执行后面的代码，而php在执行header后,会继续执行下面的代码。 如果要使php的header重定向后,不能执行后面的代码:

```php
if(...)   
header("...");   
else  
{   
...   
}   
// 也可以   

if(...)   
{header("...");break;}
```


