# [SAE福利]动态CDN

昨天听到一句话，有点睡不着（开心~~）。好吧，利用时间，写了一下javascript反向的CDN。 

原理是javascript替换用户请求的资源的地址到SAE STOR中的地址，如果不存在，则使用SAE反向抓取资源。 

SAE PHP程序inspired by SaeLayer CDN, todo list: 

1. SAE文件管理
2. SAE缓存管理
3. SAE文件安全判断
4. JS过滤某些TAG的链接替换
5. 插件化~
6. PING 检测是否使用

PHP代码目前还木有到我的标准，所以，不发了，javascript的话也略凌乱，不过目测可以看，贴一下。

```js
(function(CMD, CDN) {

    var h = document.location.hostname;

    function cdnRes(u, h, c) {
        var re = new RegExp("^(.*\/\/)(.+\.)?(" + h + ".*\.)(png|js|jpg|css)$", "i");
        if (re.test(u)) {
            var r = u.match(re);
            r[0] = '';
            if (!r[2]) {
                r[2] = '';
            } else {
                r[2] = '/' + r[2].replace('.', '') + '/';
            }
            u = r.join('').replace(h, c);
        }
        return u;
    }

    var o = document.getElementsByTagName('*')

    for (var i = 0; i < o.length; i++) {

        if (typeof(o[i].href) !== 'undefined') {
            o[i].href = cdnRes(o[i].href, h, CDN);
        } else if (typeof(o[i].src) !== 'undefined') {
            o[i].src = cdnRes(o[i].src, h, CDN);
        }
    }
})('LOAD-SOULTEARY-CDN', 'thecdn.sinaapp.com');
```

