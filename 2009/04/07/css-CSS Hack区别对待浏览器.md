# [css]CSS Hack区别对待浏览器

IE6与FireFox[FireFox在前,IE6在后]

```css
background:orange;*background:blue;
```

IE6与IE7[IE6在后]

```css
background:green !important;background:blue;
```

IE7与FireFox[IE在后]

```css
background:orange; *background:green;
```

区别FireFox,IE7,IE6[FireFox在前,IE7在中间,IE6最后]

```css
background:orange;
*background:green!important;
*background:blue;
```

区别FireFox,IE7,IE6[FireFox在前,IE7在中间,IE6最后]

```css
background:orange;
*background:green;
_background:blue;
```

## 总结

```
IE都能识别*;标准浏览器(如FF)不能识别*;      
IE6能识别*,但不识别!important,      
IE7能识别*,也能识别!important;      
FF不能识别*,但能识别!important;   
IE6支持下划线_,IE7和firefox均不支持下划线_。   
不论什么方法,顺序都是firefox的写在前面,IE7的写在中间,IE6的写在最后面。
```


