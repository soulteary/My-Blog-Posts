# 遇到的一些APACHE错误日志

前几天一直很忙碌,没顾上整理blog.然后发现各种问题.我估计让小张也很无语.
慢慢来修改吧,毕竟买的书木有看完,没有整体的印象.

<!-- more -->

错误日志:Premature end of script headers
错误解释:程序过早执行完毕.
解决方法:检查是否由于编码保存问题,以及并发链接数问题产生的问题.

错误日志:Script timed out before returning headers
错误解释:超时返回程序头部.
解决方法:设置PHP中的max_execution_time数值,如果仍然出错,尝试设置apache的Timeout的数值.

错误日志:The timeout specified has expired: ap_content_length_filter: apr_bucket_read() failed
错误解释:超时设置已过期.
解决方法:检查 snmpd.conf 配置是否正确，如 ip地址.

错误日志:Handler for x-httpd-php5 returned invalid result code 70007
错误解释:php返回码无效.
解决方法:WP程序未做try catch.


错误日志:Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel debug' to get a backtrace.
错误解释:重定向过多
解决方法:
在Apache配置文件httpd.conf中

```apache
<Directory />
    Options FollowSymLinks
    AllowOverride All                        #j将All改成 None
    Order deny,allow
    Deny from all
</Directory>
```



