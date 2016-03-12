# WordPress 简单提权工具

BLOG似乎全放水文会被鄙视，虽然这也是一篇水文。最近在做D2的票务相关的事情，管理团队blog的大神给了我一个可怜巴巴的发布权限。

而我做的插件，不管是手动开启模式需要管理员权限，还是mu-plugin也得管理员到plugin页面看一眼（可能是奇葩版本的问题）。

虽然说我有sudo，但是看过db结构的童鞋应该知道还是从wp程序来更新用户信息会容易。所以就开始了提权之路（你也可以理解为找回密码的同时，顺便提升下权限）。

## 提权开始

翻了一会文档，发觉公开api没有能直接完成的，追了一下代码，发觉WP_USER->set-role可以达成目的。于是就有了下面的这个脚本，保存内容到根目录，然后命名成你喜欢的名称，执行一下，填写你的id和新密码，回车就好了。（这里可以做到神不知鬼不觉的执行哟，用零几年的某个方法 :D）

注意: 请不要用这货来干坏事..尤其是结合最近的某些方法...

```php
<?php
/**
 * WordPress For PassWord Reset Tools
 * 2014.9.29
 * [@soulteary](http://www.soulteary.com)
 */

define('FILE_NAME', basename($_SERVER['SCRIPT_FILENAME']));

require(dirname(__FILE__) . '/wp-load.php');
require_once(dirname(__FILE__) . '/wp-admin/includes/user.php');

function makeTpl($title, $event = false){
?><!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="zh-CN">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>WordPress 芝麻开门 &rsaquo; <?= $title; ?></title>
    <?php
    wp_admin_css('wp-admin', true);
    wp_admin_css('colors-fresh', true);
    wp_admin_css('login', true);
    ?>
    <meta name='robots' content='noindex,nofollow'/>
</head>
<body class="login login-action-login wp-core-ui locale-zh-cn">
<div id="login">
    <div id="loginform">
        <h1><a href="http://cn.wordpress.org/" title="基于 WordPress">WordPress 芝麻开门</a></h1>
        <?php if ($event == 'TOKEN'): ?>
            <div id="login_error"><strong>错误</strong>：您输入的Key不正确。<br></div>
        <?php elseif ('TOOLS-INFO' == $event): ?>
            <style type="text/css">
                .login #login_error {
                    background-color: #E8F8FF;
                    border-color: #009BCC;
                }
            </style>
            <div id="login_error">请输入芝麻开门的Key。<br></div>
        <?php
        elseif ('RESET-EMPTY' == $event): ?>
            <style type="text/css">
                .login #login_error {
                    background-color: #E8F8FF;
                    border-color: #009BCC;
                }
            </style>
            <div id="login_error"><strong>提示</strong>：请将表单填写完整。<br></div>
        <?php
        elseif ('RESET-ERROR' == $event): ?>
            <div id="login_error"><strong>错误</strong>：重置密码失败。<br></div>
        <?php
        elseif ('RESET-DONE' == $event): ?>
            <style type="text/css">
                .login #login_error {
                    background-color: #E9FFE8;
                    border-color: #00CC49;
                }
            </style>
            <div id="login_error">
                <strong>成功</strong>：重置密码完成，请使用新密码登录。<br>新的登录密码：<span class="code"><?= trim($_POST['password']) ?></span></div>
        <?php endif; ?>
        <?php if ('TOOLS-INFO' == $event || 'TOKEN' == $event): ?>
        <form name="loginform" id="loginform" action="./<?= FILE_NAME ?>" method="post" style="padding-bottom: 20px;">
            <p>
                <label for="token">Sina App Engine Secret Key<br/>
                    <input type="password" name="token" id="token" class="input" value="" size="20"/></label>
            </p>

            <p class="submit">
                <input type="submit" name="wp-submit" id="wp-submit" class="button button-primary button-large"
                       value="验证"/>
            </p>
            <?php elseif ('RESET-EMPTY' == $event || 'RESET-ERROR' == $event): ?>
            <form name="loginform" id="loginform" action="./<?= FILE_NAME ?>" method="post"
                  style="padding-bottom: 20px;">
                <input type="hidden" name="token" value="<?= md5(TOKEN); ?>"/>

                <p>
                    <label for="username">帐号名称<br/>
                        <input type="text" name="username" id="username" class="input" value="" size="20"/></label>
                </p>

                <p>
                    <label for="password">新的密码<br/>
                        <input type="password" name="password" id="password" class="input" value="" size="20"/></label>
                </p>

                <p class="submit">
                    <input type="submit" name="wp-submit" id="wp-submit" class="button button-primary button-large"
                           value="修改"/>
                </p>
                <?php elseif ('RESET-DONE' == $event): ?>
                <form name="loginform" id="loginform" method="post" style="padding-bottom: 20px;">
                    <p class="submit">
                        <style type="text/css">
                            #bye-bye {
                                float: left;
                                margin: 0 0 0 86px;
                            }
                        </style>
                        <a id="bye-bye" class="button button-primary button-large" href="./wp-login.php"/>前往登录</a>
                    </p>
                    <?php endif; ?>
                </form>
    </div>
</div>
<div class="clear"></div>
</body>
</html>
<?php
exit();
}

define('TOKEN', 'zhimakaimen');
if (!isset($_POST['token']) || empty($_POST['token'])) {
    makeTpl('管理员验证', 'TOOLS-INFO');
} else {
    if ($_POST['token'] != TOKEN && ($_POST['token'] != md5(TOKEN))) {
        makeTpl('管理员验证', 'TOKEN');
    }
}

$username = trim($_POST['username']);
$password = trim($_POST['password']);

if (!isset($password) || !isset($username) || empty($username) || empty($password)) {
    makeTpl('重置帐号', 'RESET-EMPTY');
} else {

    $username = wp_slash($username);
    $user = WP_User::get_data_by('login', $username);
    $user2 = new WP_User( (int)$user->ID );
    $user2->set_role("administrator");
    wp_cache_delete((int)$user2->ID, 'users');
    wp_cache_delete($user1->user_login, 'userlogins');

    if (!$user) {
        makeTpl('重置帐号', 'RESET-ERROR');
    }
    wp_set_password($password, $user->ID);
    wp_password_change_notification($user);
    makeTpl('重置帐号', 'RESET-DONE');
}
?>
```

