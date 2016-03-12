# 最简单的mock json脚本

现在有太多方法去mock，不过当还是要连接到远程服务器上测试真正的返回的时候，如果机器上没有跑node而且有php的话，用这货来临时mock一下，或许更轻松。

```php
<?php
/**
 * Mock Json for Javascript
 *
 * @author soulteary
 * @date 2014-06-15
 */

/**
 * 请求接口字段：字符集
 */
define('charset', 'charset');

/**
 * 请求接口字段：回调函数名
 */
define('callback', 'callback');

/**
 * 请求接口字段：跨域字段
 */
define('crossDomain', 'cross-domain');

/**
 * 输出mock数据
 * 如果存在mock.json文件，则数据从mock.js中获取
 *
 * @return string
 */
function mockData()
{
    if (file_exists('mock.json')) {
        $data = json_decode(file_get_contents('mock.json'));
    } else {
        $data = Array(
            'code' => 200,
            'desc' => 'Get the default data.',
            'login' => true,
            'data' => Array(
                'name' => 'test api.'
            )
        );
    }
    return json_encode($data);
}

/**
 * 输出字符集，允许结果为gbk、gb2312、utf-8
 * 如果非法或者未设置，输出utf-8
 *
 * @return string
 */
function charset()
{
    $ret = 'utf-8';
    if (empty($_REQUEST[charset])) {
        return $ret;
    } else {
        $charset = strtolower($_REQUEST[charset]);
        if (in_array($charset, array('gbk', 'gb2312'), true)) {
            return $charset;
        } else {
            return $ret;
        }
    }
}

/**
 * 拼装json数据
 *
 * @return string
 */
function jsonGenerator()
{
    if (!empty($_REQUEST[callback])) {
        header('Content-Type: application/javascript; charset=' . charset());
        return $_REQUEST[callback] . "(" . mockData() . ");";
    } else {
        if (!empty($_REQUEST[crossDomain])) {
            header("Access-Control-Allow-Origin: *");
        };
        header('Content-type: application/json; charset=' . charset());
        return mockData();
    }
}

/**
 * 输出结果
 */
die(jsonGenerator());
```

如果你不想改动php里的data object，觉得麻烦，那么直接改动json好了，你或许会问，那我为啥不直接访问一个json呢，答：
 
1. 你或许需要一个callback包装这个结果；
2. 你或许期望这个json允许跨域请求；
3. 你或许期望这个json可以自定义header编码...

```js
{
    "data": 1,
    "w": "测试"
}
```

代码很简单，就不过多描述了。
