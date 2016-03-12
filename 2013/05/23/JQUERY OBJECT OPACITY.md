# JQUERY OBJECT OPACITY

[![2013-05-23_031655](https://attachment.soulteary.com/2013/05/23/2013-05-23_031655.png "2013-05-23_031655")](https://attachment.soulteary.com/2013/05/23/2013-05-23_031655.png)

更新了一个插件，**JQUERY OBJECT OPACITY** A jQuery plugin that changes element(s) opacity, support delegate in javascript. 

一个简单的JQEURY插件，根据传入对象的数量来进行选择方案，对元素进行事件绑定。

当设置多个对象的时候，可以自行选择使用事件代理，或者是逐一绑定。

- **GIT DEMO:**[http://soulteary.github.io/jQuery-Object-Opacity/](http://soulteary.github.io/jQuery-Object-Opacity/)
- **SAE DEMO:**[http://thecdn.sinaapp.com/page/demo/jQuery-Object-Opacity/](http://thecdn.sinaapp.com/page/demo/jQuery-Object-Opacity/)

## 用法示例：

```js
//对于有一堆相同属性的元素
$('.obj-opacity').objOpacity({
    warp:'#warp',//如果不想使用事件代理,就不要使用这个参数,设置false
    hover:'effect',
    event:'mouseover',
    bevent:'mouseout',
    focus:1,
    blur:.1,
    speed:600
});

//对于只有一个属性的元素
$('.single').objOpacity({
    hover:'effect',
    event:'click',
    bevent:'mouseout',
    focus:1,
    blur:.1,
    speed:600
});
```


