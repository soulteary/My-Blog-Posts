# JS - 标题定时闪动

标题闪动是一个比较老的需求了，今天有童鞋需要，写了一段简单的实现。

```js
var blinkTitle = function (option) {
    var title = null;
    var newTitle = null;
    var handle = null;
    var state = false;
    var interval = null;

    if (option) {
        newTitle = option.newTitle ? option.newTitle : '';
        title = option.title ? option.title : document.title;
        interval = option.interval ? option.interval : 600;
    } else {
        newTitle = '';
        title = document.title;
        interval = 600;
    }

    function start() {
        if (state === true) {
            document.title = newTitle;
            state = false;
        } else {
            document.title = title;
            state = true;
        }
        handle = setTimeout(arguments.callee, interval);
    }

    function stop(option) {
        clearTimeout(handle);
        setTimeout(function () {
            console.log(option)
            if (typeof option === "string") {
                document.title = option;
            } else {
                document.title = title;
            }
        }, 200);
    }

    return {
        start: start,
        stop: stop
    }
}();

/**
 * 开始标题闪烁
 */
blinkTitle.start();
//可以传入几个可选的参数
blinkTitle.start({
    title: '设置新的文档默认标题内容',
    newTitle: '设置新的闪动标题内容',
    interval: 1000
});

/**
 * 停止标题闪烁
 */
blinkTitle.stop();
//如果你想让页面标题为一个新的内容的话，可以传入一个字符串
blinkTitle.stop('新的标题内容');
```
