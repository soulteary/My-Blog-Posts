# 带超时功能的get_headers

代码比较多，但是比较简单，一眼就看明白的，so，文字尽量少写了。

因为众所周知的网络原因，gavatar也开始越来越慢，写了一个小东西来解决这个问题，过程中遇到了get_headers这个函数，甚是忧伤，记录下来，以免后来人踩坑。 更新记录，函数稍微改了一下，返回值基本和之前序列化后的结果一致，暂时没考虑支持子项也支持数组等（考虑细节性能，还想把没用的http头砍掉....）

需求很简单：获取图片的head信息。

调试程序的时候发现这个函数的调用很缓慢，即使绑定ip，有时候都能蹦到20多秒。

寻思这个事情还是该加个超时吧，但是看官方文档，给出的导出函数接口如下：

```php
array get_headers ( string $url [, int $format = 0 ] )
```

你没有看错，这个东西没有超时接口...

上GitHub翻看源码，期望可以用他的底层实现来重新实现一套：

- https://github.com/php/php-src/blob/88ca46d92bc1c426e7c7f7313f0fd2b7dcc33cf6/ext/standard/url.c#L710

```c++
/* {{{ proto array get_headers(string url[, int format])
   fetches all the headers sent by the server in response to a HTTP request */
PHP_FUNCTION(get_headers)
{
	char *url;
	size_t url_len;
	php_stream_context *context;
	php_stream *stream;
	zval *prev_val, *hdr = NULL, *h;
	HashTable *hashT;
	zend_long format = 0;

	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s|l", &url, &url_len, &format) == FAILURE) {
		return;
	}

	/** 省略其他一堆... **/
}
/* }}} */
```

但是很不幸的是`zend_parse_parameters`和`ZEND_NUM_ARGS`也都没有PHP版的导出函数。

于是造轮子开始：

```php
function get_url_headers($url, $timeout = 10)
{
    $ch = curl_init();

    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_HEADER, true);
    curl_setopt($ch, CURLOPT_NOBODY, true);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_TIMEOUT, $timeout);

    $data = curl_exec($ch);
    $data = preg_split('/\n/', $data);

    $data = array_filter(array_map(function ($data) {
        $data = trim($data);
        if ($data) {
            $data = preg_split('/:\s/', trim($data), 2);
            $length = count($data);
            switch ($length) {
                case 2:
                    return array($data[0] => $data[1]);
                    break;
                case 1:
                    return $data;
                    break;
                default:
                    break;
            }
        }
    }, $data));

    sort($data);

    foreach ($data as $key => $value) {
        $itemKey = array_keys($value)[0];
        if (is_int($itemKey)) {
            $data[$key] = $value[$itemKey];
        } elseif (is_string($itemKey)) {
            $data[$itemKey] = $value[$itemKey];
            unset ($data[$key]);
        }
    }

    return $data;
}
```

对比最后结果，原版又是蛮长的等待，不知道校验啥去了（没继续追代码了，有兴趣的童鞋可以去跟下玩）：

```php
Array
(
    [0] => HTTP/1.0 302 Found
    [Accept-Ranges] => bytes
    [Cache-Control] => max-age=300
    [Content-Type] => Array
        (
            [0] => text/html; charset=utf-8
            [1] => text/html; charset=utf-8
        )

    [Date] => Array
        (
            [0] => Fri, 12 Dec 2014 15:35:40 GMT
            [1] => Fri, 12 Dec 2014 15:35:43 GMT
        )

    [Expires] => Fri, 12 Dec 2014 15:40:40 GMT
    [Last-Modified] => Wed, 11 Jan 1984 08:00:00 GMT
    [Link] => <http://www.gravatar.com/avatar/[省略...]?s=42&d=http%3A%2F%2F[省略...]&r=G>; rel="canonical"
    [Location] => http://i2.wp.com/[省略...]
    [Server] => Array
        (
            [0] => ECS (oxr/838B)
            [1] => nginx
        )

    [Source-Age] => 85
    [Via] => 1.1 varnish
    [X-Cache] => 302-HIT
    [X-Varnish] => 1470255088 1470006304
    [Content-Length] => 0
    [Connection] => Array
        (
            [0] => close
            [1] => close
        )

    [1] => HTTP/1.1 504 Gateway Timeout
)
```

轮子版返回(瞬间返回，两者内容略有不同，你仔细看就能发现一些有趣的地方了）：

```php
Array
(
    [0] => HTTP/1.1 302 Found
    [Accept-Ranges] => bytes
    [Via] => 1.1 varnish
    [Cache-Control] => max-age=300
    [Server] => ECS (oxr/838B)
    [Content-Type] => text/html; charset=utf-8
    [X-Varnish] => 1470255088 1470006304
    [Date] => Fri, 12 Dec 2014 20:31:02 GMT
    [Location] => http://i2.wp.com/[省略...]
    [Expires] => Fri, 12 Dec 2014 20:36:02 GMT
    [Source-Age] => 85
    [Last-Modified] => Wed, 11 Jan 1984 08:00:00 GMT
    [X-Cache] => 302-HIT
    [Link] => <http://www.gravatar.com/avatar/[省略...]?s=42&d=http%3A%2F%2F[省略...]&r=G>; rel="canonical"
    [Content-Length] => 0
)
```
