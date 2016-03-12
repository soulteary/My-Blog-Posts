# [PHP]WordPress之DashBoard改进

WordPress的DashBoard其实不错,可惜自定义程度不高,况且一般用户[非管理员]登陆后没必要显示太多的信息,很多人都有删除里面代码的想法，在我看来，大可不必，看完下面这篇文章你可以实现：

1. 根据用户类型，控制DashBoard的输出。
2. 提速后台的加载。

打开wp-admin目录后,我们可以看到index.php文件,使用支持UTF-8的编辑器打开后,
将

```php
<?php screen_icon(); ?>
```

与

```php
<?php require(ABSPATH . ‘wp-admin/admin-footer.php’); ?>
```

之间的代码修改如下：

```php
<?php
//只有管理员显示不同控制面板
if(current_user_can(‘level_10′))
{
?>
<h2><?php echo esc_html( $title ); ?></h2>
?<div id="dashboard-widgets-wrap">
??<?php wp_dashboard(); ?>
??<div></div>
?</div><!– dashboard-widgets-wrap –>
</div><!– wrap –>
<??
}
else
{
?>
<h2>一般用户看到的标题</h2>
<div id="dashboard-widgets-wrap">
一般用户看到的提示信息
<div></div>
</div><!– dashboard-widgets-wrap –>
</div><!– wrap –>
<??
}
?>
```

