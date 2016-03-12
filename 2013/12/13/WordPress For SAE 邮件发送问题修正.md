# WordPress For SAE 邮件发送问题修正

由于一直没有修改密码的需求以及发送邮件的需求，没有及时测试以及修改，对给wp fans造成的不便在这里深表抱歉。

接下来的新版本将会集成并自动开启邮件插件，如果你有发送邮件的需求，只需要登录并设置一下你的邮箱帐号密码即可。

下面记录两件事情：

第一件，有用户反馈找回密码功能发送的邮件会在地址结尾处多添加一个右尖括号（>），于是使用QQ Mail测试了一下，看到下面的情况。

[![2013-12-12_162812](https://attachment.soulteary.com/2013/12/13/2013-12-12_162812.png "2013-12-12_162812")](https://attachment.soulteary.com/2013/12/13/2013-12-12_162812.png)

```php
$message .= '<' . network_site_url("wp-login.php?action=rp&key=$key&login=" . rawurlencode($user_login), 'login') . ">\r\n";
```

解决方法也很简单，把这句字符串拼接前后的尖括号去掉，WP只是为了美观顺手加上的吧，结果QQ Mail智能了一下，但是没有正确的匹配结束。

```php
$message .= network_site_url("wp-login.php?action=rp&key=$key&login=" . rawurlencode($user_login), 'login') . "\r\n";
```

再次发送邮件，果然没有问题了。

[![2013-12-12_164506](https://attachment.soulteary.com/2013/12/13/2013-12-12_164506.png "2013-12-12_164506")](https://attachment.soulteary.com/2013/12/13/2013-12-12_164506.png)

第二件，简单对wp_mail_smtp插件进行汉化和修改，把不支持的方法去掉，以及默认写出QQ Mail邮箱服务器的一些参数。具体修改见GitHub。

设置方式也很简单，在登录网站后，访问设置菜单中的邮箱设置。

[![2013-12-12_155723](https://attachment.soulteary.com/2013/12/13/2013-12-12_155723.png "2013-12-12_155723")](https://attachment.soulteary.com/2013/12/13/2013-12-12_155723.png)

设置完毕之后，点击发送测试邮件，不出意外你的邮箱会收到测试邮件，如果没有收到，请仔细检查上面填写的内容是否正确。

[![2013-12-12_161406](https://attachment.soulteary.com/2013/12/13/2013-12-12_161406.png "2013-12-12_161406")](https://attachment.soulteary.com/2013/12/13/2013-12-12_161406.png) 

至此，两个问题都解决了。 如果对WordPress For SAE 还有任何使用问题，欢迎反馈。 更新细节，[点击查看](https://github.com/soulteary/wordpress-for-sae/commit/dfbc8a48d9a38f96fa9fb74519f75d4235ef660e)。

