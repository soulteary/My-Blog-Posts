# 自动调整zindex数值不超过最大值

话说今天遇到一个需求，因为项目战线拉长之后，完成不是依靠一个两个同学的，可能很多，或由于历史原因，或由于编写者的代码习惯不同。

在细节上会出现冲突，比如使用javascript设置不同元素的zindex，javascript创建一个模态的覆盖层后，应该是zindex仅次于上一级要显示的内容。

[![showmaskbug](https://attachment.soulteary.com/2012/09/28/showmaskbug.png "showmaskbug")](https://attachment.soulteary.com/2012/09/28/showmaskbug.png)

但是如图所示，在showMask创建一个覆盖的浮层之后，有至少两个元素的zindex穿破了这个浮层。 如果强制更新css文件的话，未免工程量颇大，尤其是你要检查每一个带有zindex的元素（当然编写前应该有完整的zindex使用规范，可是实际操作的时候...）

言归正传，那么javascript就派出用场了。 在调用创建浮层的函数的尾部添加这个函数，即可把zindex设置特别大的元素调整的比浮层元素的zindex小10个数值。 代码十分简单，就不描述实现了。

需要注意的是，如果有特别需求，可以在关闭浮层的时候，把之前的元素的数值还原回去，图省事，使用全局变量即可。

```
function adjustzIndex(maxValue) {
    var a = document.getElementsByTagName('body')[0].getElementsByTagName('*');
    for (var i = 0; i < a.length; i++) {
        if ('SCRIPT' != a[i].nodeName && a[i].style.zIndex) {
            if (maxValue <= a[i].style.zIndex) {
                a[i].style.zIndex = maxValue - 10;
            }
            //用于调试输出zindex数值
            //console.log('zIndex: ',a[i].style.zIndex,'Index: ',i,' >>> ','Element: ',a[i],'Type:',a[i].nodeName);                    
        }
    }
}
```


