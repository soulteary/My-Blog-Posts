# WP插件 - Open Fonts替换

最近由于Google Open Fonts的众所周知的大姨妈问题，WordPress博客普遍出现了访问很慢的状况。 

写了一个小插件，使用奇虎360的开放字体服务来替换Google的开放字体服务，插件任何问题，欢迎反馈。

> === Replace Google Fonts === Contributors: soulteary Tags: Qihu Fonts, Qihu Web Fonts, 360 Fonts, 360 Web Fonts, Google Fonts, Google Web Fonts Requires at least: 3.5 Tested up to: 3.8 Stable tag: 1.0 Use Qihoo 360 Open Fonts Service to replace Google's. == Description == [Plugin homepage](http://www.soulteary.com/2014/06/08/) | [Plugin author](http://www.soulteary.com/) There are some problem with Google in china, the Google Fonts make the website too slow to open, so we can use Qihoo 360's Web Fonts replace the Google's. [Project GitHub](https://github.com/soulteary/Replace-Google-Fonts). == Installation == 1\. Upload `replace-google-fonts` folder to the `/wp-content/plugins/` directory 2\. Activate the plugin through the 'Plugins' menu in WordPress

```php
/**
 * Plugin Name: Replace Google Fonts
 * Plugin URI:  http://www.soulteary.com/2014/06/08/replace-google-fonts.html
 * Description: Use Qihoo 360 Open Fonts Service to replace Google's.
 * Author:      soulteary
 * Author URI:  http://www.soulteary.com/
 * Version:     1.0
 * License:     GPL
 */

/**
 * Silence is golden
 */
if (!defined('ABSPATH')) exit;

class Replace_Google_Fonts
{

    /**
     * init Hook
     *
     */
    public function __construct()
    {
        add_filter('style_loader_tag', array($this, 'ohMyFont'), 888, 4);
    }

    /**
     * Use Qihoo 360 Open Fonts Service to replace Google's.
     *
     * @param $text
     * @return mixed
     */
    public function ohMyFont($text)
    {
        return str_replace('//fonts.googleapis.com/', '//fonts.useso.com/', $text);
    }
}

/**
 * bootstrap
 */
new Replace_Google_Fonts;
```

插件下载：[Replace-Google-Fonts](https://attachment.soulteary.com/2014/06/Replace-Google-Fonts.zip)

