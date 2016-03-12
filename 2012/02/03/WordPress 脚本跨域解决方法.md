# wordpress脚本跨域解决方法

说到跨域问题，我想很多人都知道，所以不做太详细解释了。

针对跨域，wordpress进行了过滤，在定时发布的一篇日志中，我有提到如何去掉wordpress的版本号，文章的修改方法是没有错误的，但是在实践的过程中，我还将静态资源移动到了子域名中，所以呢，资源就跨域访问了。

那么如何解决静态脚本跨域使用呢，在wordpress中有3个专门的函数**wp_deregister_script**,**wp_register_script**以及**wp_enqueue_script**

```php
function fir_scripts_method() {
    wp_deregister_script( 'jquery' );
    wp_register_script( 'jquery', 'http://yoururl/wp-includes/js/jquery/jquery.js');
    wp_enqueue_script( 'jquery' );
}
add_action('wp_enqueue_scripts', 'fir_scripts_method_jquery');
```

将代码添加到你的后台模版或者post.php合适位置,即可.

这么一来,前台依旧是静态化的子域名资源,而后台重新使用函数注册并安全输出脚本,跨域神马的就再见了。

