# JS 批量删除豆瓣想看的电影

计划把社交帐号都整理一下,发现豆瓣里的电影已经无法正常使用了,因为某个时候随意添加了大量的想看的电影.

而现在，我想把他们都清空，简单的做法的话，肯定不是手动蠢蠢的点击，那么就需要写一段简单的脚本了。

一段简单的code,可以自动提交请求,将你想看的内容删除掉,如果你页面不多,自己点点就ok了,如果很多的话,用node抓页面中所有的翻页的链接, 然后再提交吧,下面是执行代码,比较简单:

```js
$('a.d_link').each(function(k, v) {
    var id = $(v).attr('rel');
    var vcode = $('.more-items a[href*=logout]').attr('href').split('ck=')[1];
    $.ajax({
        url: 'http://movie.douban.com/j/mine/j_cat_ui',
        type: 'POST',
        contentType: 'application/x-www-form-urlencoded; charset=UTF-8',
        data: {
            sid: id,
            ck: vcode
        },
        success: function(e) {
            console.log(e);
        }
    })
});
```

豆瓣在做检查的时候,额外设置需要设置**contentType**为`application/x-www-form-urlencoded; charset=UTF-8`, 这个api需要的参数，一个为sid（顾名思义，删除链接ref属性内的hash），另一个为ck（猜测是check的意思吧，取值可以从登出的地方拿，或许是current key? 这个细节不必在意，反正达到删除的目的鸟。）

---EOF---


