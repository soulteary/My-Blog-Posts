# WP插件 - Google Libs替换

同Open Fonts，Open Libs也躺枪，网上的解决方案是在输出之前替换掉所有的Google域下的文件，感觉不妥，还是使用钩子来实现吧，如果你说，我的主题里有写死的脚本引用，怎么办？

答：手动替换(提供插件的方式就是为了一劳永逸，写死的代码提高效率有限而束缚了维护)。

官方一般会告诉开发者如此使用脚本库定义：

```php
function my_scripts_method() {
    wp_deregister_script( 'jquery' );
    wp_register_script( 'jquery', 'http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js');
    wp_enqueue_script( 'jquery' );
}

add_action('wp_enqueue_scripts', 'my_scripts_method');
```

针对上面这样重新注册脚本库的方式，使用下面的插件代码即可完美解决Google脚本库因网络问题拖慢网站速度。

```php
<?php
/**
 * Plugin Name: Replace Google Libs
 * Plugin URI:  http://www.soulteary.com/2014/06/15/replace-google-libs.html
 * Description: Use Qihoo 360 Open Libs Service to replace Google's.
 * Author:      soulteary
 * Author URI:  http://www.soulteary.com/
 * Version:     1.0
 * License:     GPL
 */

/**
 * Silence is golden
 */
if (!defined('ABSPATH')) exit;

class Replace_Google_Libs
{

    /**
     * init Hook
     *
     */
    public function __construct()
    {
        add_filter('wp_print_scripts', array($this, 'ohMyScript'), 1000, 1);
    }

    /**
     * Use Qihoo 360 Open Libs Service to replace Google's.
     */
    public function ohMyScript()
    {
        global $wp_scripts;
        if($wp_scripts && $wp_scripts->registered){
            foreach ($wp_scripts->registered as $libs){
                $libs->src = str_replace('//ajax.googleapis.com', '//ajax.useso.com/', $libs->src);
            }
        }
    }
}

/**
 * bootstrap
 */
new Replace_Google_Libs;
```

插件下载：

- [Replace-Google-Libs](https://attachment.soulteary.com/2014/06/Replace-Google-Libs.zip) 
- GitHub: https://github.com/soulteary/Replace-Google-Libs
