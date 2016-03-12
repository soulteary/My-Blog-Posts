# [js]只能输入数字

```js
isNumber = function(e) {
    if ($.browser.msie) {
        if (((event.keyCode > 47) && (event.keyCode < 58)) ||
            (event.keyCode == 8)) {
            return true;
        } else {
            return false;
        }
    } else {
        if (((e.which > 47) && (e.which < 58)) ||
            (e.which == 8)) {
            return true;
        } else {
            return false;
        }
    }
}
```

```html
<input type="text" onkeypress="javascript:return isNumber(event);">
```

```js
// IE的event还会冒泡，阻止IE事件冒泡，在js中添加
event.cancelBubble = true;
```


