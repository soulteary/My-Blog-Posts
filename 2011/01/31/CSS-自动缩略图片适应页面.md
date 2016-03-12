# [CSS]自动缩略图片适应页面

重写的时候，翻CSS看到一段内容，感觉很古老了，但是或许什么时候写简单页面会用上，留下来吧。

```css
.post_img {  
    max-width:600px;height:auto;cursor:pointer;  
    border:1px dashed #4E6973;padding: 3px;  
    zoom:expression( function(elm) {  
        if (elm.width>560) {  
            var oldVW = elm.width; elm.width=560;  
            elm.height = elm.height*(560 /oldVW); 
        } 
        elm.style.zoom = '1'; 
    }(this));
} 
```


具体数值，可以修改。比如最大宽度600，适应宽度560等。

