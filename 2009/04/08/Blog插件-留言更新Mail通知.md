# [Blog插件]留言更新Mail通知

很多时候,blog上的留言不能及时的回复造成了人气的流失,所以做了一个小插件。通过探针查看服务器的组建状况,得到服务器支持Mail()函数,于是使用Mail函数来写这个小插件。

```php
www.promiseforever.com 苏洋
//修正时间,如果服务器Date获取时间正常就不需要了
$MailTimeStamp=date("g")+8;
$MailTimeStamp=date("Y-m-d H:i:s");
//邮件主题
$MailSubject="网站用户: $username 留了一条评论# $articleid";

//邮件正文
$MailMessage="留言时间: $MailTimeStamp
留言用户: $username
留言序号: $articleid
留言地址: $onlineip
留言内容: $content
联系方式: $url
是否显示: $visible";

//判断用户是否为游客
if (!$sax_uid){
//邮件主题
$MailSubject="游客[$onlineip] 留了一条评论# $articleid";
//邮件正文
$MailMessage="留言时间: $MailTimeStamp
留言序号: $articleid
留言地址: $onlineip
留言内容: $content
联系方式: $url
是否显示: $visible";
}
//发送邮件
$mailRe .= (false !== @mail($MailReceiver, $MailSubject, $MailMessage,$MailHeaders))?"完成":"失败";
?>
```

[gallery]

