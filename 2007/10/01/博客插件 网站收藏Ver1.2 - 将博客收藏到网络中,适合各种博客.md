# 博客插件 网站收藏Ver1.2 - 将博客收藏到网络中，适合各种博客

今天随意闲逛到一个博客，发现它有一个文章收藏功能，于是把在其脚本的基础上稍作修改，并把最近火的不行的QQ书签和百度搜藏添加了进去。

## 正文

sa-blog修改方法：

打开正在使用的模板中的index.php 找到

```html

## 发表评论

```

将其修改成：

```html
<a name="PromiseForever.com_soucang"></a>

## 收藏本文到网摘

<a name="viewcomment"></a><a title="Made by 花好月圆. <a href=" http:="" www.promiseforever.com"="">www.promiseforever.com</a>" href="[http://www.promiseforever.com/& lt;/a>">可以将本文收藏到:](http://www.promiseforever.com/)
<a name="addcomment"></a>

## 发表评论

```

然后下载脚本文件和收藏图标， 将代码稍作修改传至你的ftp空间~ [本站开启防盗链功能，所以请将图片保存在自己的空间或者是博客上面] 本文方法支持各种博客，如：sa-blog、o-blog、sina-blog、sohu-blog... 在线申请博客如果要有此功能，只需要在博客发布文章的时候，加入html源代码

```html

<div>
<a name="PromiseForever.com_soucang"></a>

## 收藏本文到网摘

<a name="viewcomment"></a>[可以将本文收藏到:](http://www.promiseforever.com/ "Made by 花好月圆. www.promiseforever.com")
</div>

```

附件:

- [download id="22"]

