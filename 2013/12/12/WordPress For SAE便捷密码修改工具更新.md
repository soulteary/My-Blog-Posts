# WordPress For SAE便捷密码修改工具更新

之前更新WordPress For SAE的时候打包了一个工具，wp-reset-password，有用户反馈工具使用有问题，于是更新之。 模板部分就不贴了，有兴趣的童鞋可以翻看[GitHub](https://github.com/soulteary/wordpress-for-sae)，或者直接前往wp4sae.org [wp4cloud.sinaapp.com]下载一份代码查看。 如果你使用的是非SAE版的WordPress一样可以使用下面的代码，将SAE_SECRETKEY替换你觉得靠谱的字符串即可。

```php
#makeTpl 即模板函数，见源文件。
define('TOKEN', SAE_SECRETKEY);
if(!isset($_POST['token'])||empty($_POST['token'])){
    makeTpl('管理员验证');
}else{
    if($_POST['token'] != TOKEN){
        makeTpl('管理员验证','TOKEN');
    }
}

$username = trim($_POST['username']);
$password = trim($_POST['password']);

if (!isset($password)||!isset($username)||empty($username)||empty($password)) {
    makeTpl('重置帐号','RESET-EMPTY');
} else {

    $username = wp_slash( $username );
    $user = WP_User::get_data_by('login', $username);

    if(!$user){
        makeTpl('重置帐号','RESET-ERROR');
    }
    wp_set_password($password, $user->ID);
    wp_password_change_notification( $user );
    makeTpl('重置帐号','RESET-DONE');
}
```

### 相关链接：

1.  [文档源码](https://github.com/soulteary/wordpress-for-sae/blob/master/wp-reset-password.php)
2.  [修改细节](https://github.com/soulteary/wordpress-for-sae/commit/a36a67b258903bee7fe5b58ad55ab23763a9da7d)

