# [PHP]WordPress之找回密码改进

或许国内的人的素质高一点吧，WordPress在制作的时候没有考虑到用户恶意提交找回密码，我记得之前有小菜帮我找回密码来着，再次贴出来解决的方案…
对于一些用户，我们不需要找回密码，或者我们可以使用其他的表单变量来进行特别的用户的密码找回。
在此我只写出不找回密码的方法。
依旧使用支持UTF-8的编辑器打开 wp-login.php文件，然后向下查找，

```php
<?php
// redefining user_login ensures we return the right case in the email
 $user_login = $user_data->user_login;
 $user_email = $user_data->user_email;
?>
```

在代码后添加一个判断即可,比如

```php
<?php
if ("<a href="mailto:soulteary@qq.com&quot;==$user_email">soulteary@qq.com"==$user_email</a>)
  {
   die();
  }
?>
```

你也可以给一个空的修改成功的提示，但是却不重置密码.
上面的方法只能判断单独的字串，甚不方便，还好，php对于数组的支持超出我们的想象。

```php
<?php
//防止人品有问题的垃圾恶作剧
 //检测是否有邮件列表中的邮件
 if (in_array($user_email,array(<a href="mailto:’soulteary@qq.com’">’soulteary@qq.com’</a>, <a href="mailto:’soulteary@gmail.com’">’soulteary@gmail.com’</a>)))
 {
   die(‘joke~?’);
 }
 //检测是否有用户列表中的ID
 if (in_array($user_email,array(‘花好月圆’, ‘苏洋’)))
 {
   die(‘joke~?’);
 }
?>
```

