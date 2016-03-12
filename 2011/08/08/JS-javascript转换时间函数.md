# [JS]javascript转换时间函数

网上找到的一个函数,语义是很明确,但是效率不是很高,不过javascript本来就是解释型的语言...效率的话,某种层次吧

```js
function MillisecondToDate(msd)
 {
    var time = parseFloat(msd) / 1000;
    if (null != time && "" != time)
 {
        if (time > 60 && time < 60 * 60)
 {
            time = parseInt(time / 60.0) + "分钟" + parseInt((parseFloat(time / 60.0) -
                parseInt(time / 60.0)) * 60) + "秒";
        }
        else if (time >= 60 * 60 && time < 60 * 60 * 24)
 {
            time = parseInt(time / 3600.0) + "小时" + parseInt((parseFloat(time / 3600.0) -
                parseInt(time / 3600.0)) * 60) + "分钟" +
                parseInt((parseFloat((parseFloat(time / 3600.0) - parseInt(time / 3600.0)) * 60) -
                parseInt((parseFloat(time / 3600.0) - parseInt(time / 3600.0)) * 60)) * 60) + "秒";
        }
        else
 {
            time = parseInt(time) + "秒";
        }
    }
    return time;
}
```

