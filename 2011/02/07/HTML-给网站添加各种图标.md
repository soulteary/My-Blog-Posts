# [HTML]给网站添加各种图标

今天上线,看到WEB QQ逆袭归来,习惯性的先看源代码,看到很多META,很多都是windows mobile支持的属性.看来智能手机确实是一个不可忽视的客户端份额。

head区域有一个地方感觉写的很好，摘下来，实现就是根据不同的浏览器和功能，将网站的图标缓存到客户端上。

首先这行是W3C标准的icon写法,所有支持WEB标准的浏览器都可以看到并且解释。
当然有的浏览器默认会直接读取你网站根目录的favicon.ico
如何制作稍后解释。

```html
<link rel="icon" href="./favicon.ico" type="image/x-icon">
```

下面这行是为MS的IE准备的，市场浏览器是MS做大，人有理由建立自己的标准，不是么。

```html
<link rel="shortcut icon" href="./favicon32.ico" type="image/x-icon">
```

FF,IE支持书签添加图标,但是你要加上下面这句

```html
<link rel="bookmark" href="./favicon.ico" type="image/x-icon">
```

Iphone越来越多的今天,不为apple添加一点支持,似乎不行..所以加上这句吧。

```html
<link rel="apple-touch-icon" href="./favicon.png">
```

接下来是完整的代码。

```html
<link rel="icon" href="./favicon.ico" type="image/x-icon">
<link rel="shortcut icon" href="./favicon32.ico" type="image/x-icon">
<link rel="bookmark" href="./favicon.ico" type="image/x-icon">
<link rel="apple-touch-icon" href="./favicon.png">
```

接下来是具体的实践过程。

推荐使用Axialis IconWorkshop 和adobe photoshop进行操作，
现在ps建立一个72x72的画布,然后尽你可能做的简约一点，这个是为了iphone和其他icon做的模板。
觉得合适了就保存为“储存为web和设备所用格式”。
 [![建立模板](https://attachment.soulteary.com/2011/02/07/2011-02-07_142412.png "建立模板")]
 
接着你需要打开IconWorkshop，首先使用软件打开刚刚保存好的favicon.png,并选择“从图像新建windows图标”。
首先建立favicon32.ico,选项为尺寸32x32，颜色为XP.
[![建立icon](https://attachment.soulteary.com/2011/02/07/2011-02-07_143102.png "建立icon")](https://attachment.soulteary.com/2011/02/07/2011-02-07_143102.png)

搞定后，保存为favicon32.ico

然后继续打开刚刚的图片,建立一个16x16,颜色为256的icon
[![建立16x16图标](https://attachment.soulteary.com/2011/02/07/2011-02-07_143311.png "建立16x16图标")](https://attachment.soulteary.com/2011/02/07/2011-02-07_143311.png) 


