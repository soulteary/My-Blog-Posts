# 更靠谱一些的正则表达式验证JS URL

看草稿的时候，看到一段代码，想到之前被不知道谁随手抄来的正则坑的内牛满面...

下面是一段相对全面的用例测试，可以考虑使用，当然，如果你的要求很简单，用网上传的各种简单版，cover到自己的用例就好。

```js
/**
 * 正则表达式判断网址是否有效
 */

(function () {
    "use strict";

    var urlDict = [
        //Bad Case
        'www.baidu.com',                      //常规网址，未带协议头的地址
        'w.baidu.com',                        //常规网址，短子域名
        'baidu.com',                          //常规网址，仅有主域名
        '测试.com',                           //非常规合法网址，中文域名不在参考之列
        '1.2',                                //错误域名
        '  WWWW ',                            //无效字符串
        '111测试',                            //无效字符串
        //Correct Case
        'http://baidu.com',                   //常规网址，仅有主域名
        'http://www.baidu.com',               //常规网址，带子域名
        'https://www.baidu.com/',             //常规网址，使用https协议头，带根目录
        'http://www.baidu.com/api',           //常规网址，有一级目录下资源
        'http://www.subdomain.baidu.com/index/subdir',      //常规网址，多级子域名，多级目录
        'http://www.www.subdomain.baidu.com/index/subdir/', //常规网址，多级子域名，多级目录，目录地址闭合
        'http://io.io'                        //非常规网址，多级子域名，多级目录，目录地址闭合
    ];

    // 建议的正则
    function isURL(str) {
        return !!str.match(/(((^https?:(?:\/\/)?)(?:[-;:&=\+\$,\w]+@)?[A-Za-z0-9.-]+|(?:www.|[-;:&=\+\$,\w]+@)[A-Za-z0-9.-]+)((?:\/[\+~%\/.\w-_]*)?\??(?:[-\+=&;%@.\w_]*)#?(?:[\w]*))?)$/g);
    }

    // 不知道谁写的简单版的坑爹正则
    function badRegFn(str) {
        return !!str.match(/(http[s]?|ftp):\/\/[^\/\.]+?\..+\w$/g);
    }

    // 测试用例覆盖
    (function () {
        var ret = {};

        var collect = function (link) {
            var obj = {}, fnList = [isURL, badRegFn];
            for (var i = 0, j = fnList.length; i < j; i++) {
                var fn = fnList[i];
                obj[fn.name] = fn.call(null, link);
            }
            return obj;
        };

        for (var i = 0, j = urlDict.length; i < j; i++) {
            ret[urlDict[i]] = collect(urlDict[i]);
        }

        console.log(ret) , console.table(ret);
    }());

}());
```
