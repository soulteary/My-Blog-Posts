# DOM获取集合元素效率比较

很好奇JQUERY选择器,HTML-DOM,DOM-CORE三个DOM操作起来的速度..

<!-- more -->

执行**jQuery('form');**发现返回数组.

执行**document.forms;**一样返回数组. 
**document.getElementsByTagName("form");**返回对象..

猜测HTML-DOM快于DOM-CORE,JQUERY 不确定(后来想到,这个东西是warp的javascript,理论应慢) HTML的根是WINDOWS,DOM-CORE的根是DOCUMENT.

使用如下代码进行测试,因为语句简单,所以需要增大循环次数来进行比较.

```
<script>
var START=new Date().getTime();
for (i=0;i<1000;i++){

//测试语句

}
var END=new Date().getTime() - START;
alert("total:"+END+"ms");
</script>
```

测试DOM-CORE的语句为: **document.getElementsByTagName("form");** 

测试HTML-DOM的语句为: **document.forms;** 

JQUERY-DOM语句为: **jQuery('form');** 

## 执行结果

- 12ms 10ms 7ms
- 6ms 3ms 6ms
- 108ms 90ms 89ms

## 结论

所以...HTML-DOM > DOM-CORE > JQUERY-DOM

