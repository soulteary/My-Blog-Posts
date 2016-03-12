# [css]简单改变网页颜色

还记得汶川地震么，国内的网站纷纷失去了颜色。当时主流的做法是用CSS的滤镜。 现在我们快过年，为何不通过一句简单的CSS滤镜语句，把网页变成红色，黄色等亮丽的颜色呢。 如果要改变整个表单的颜色

```css
body{filter:Gray;}
```

如果要改变图片的颜色

```css
img{filter:Gray;}
```

如果只是要某个div或者某个对象变色，只需要在对应的div class中添加

```css
{filter:Gray;}
```

具体例子：

```css
.ChangeColor{filter:Gray;}
```

```html
<Img Src=Img.gif Class="ChangeColor">
<div class="ChangeColor">要变色的内容</div>
```


