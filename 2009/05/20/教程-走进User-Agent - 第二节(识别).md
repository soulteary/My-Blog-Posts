# [教程]走进User-Agent - 第二节(识别)

下面是一些常见的浏览器的User-Agent标识，其实不光是浏览器会有标识，只要是使用XMLHTTP Web Api进行网络活动的application都会有标识的，比如你用迅雷，快车，电驴等工具下载的时候，是不是常常出现一个“引用页”呢，这些工具会模仿一些正常的用户浏览所形成的标识来通过HTTP Server的检查。[所以说，彻底的防止资源被盗链的途径就是制作专用下载客户端，并且通过一些小手段来造成无法模拟的效果(非常简单，不过说出来就不好玩了)]

```text

IE 5.5:
Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.1; .NET CLR 2.0.50727; InfoPath.1; .NET CLR 1.1.4322)

IE 6:
Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 2.0.50727; InfoPath.1; .NET CLR 1.1.4322)

IE 7:
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; .NET CLR 2.0.50727; InfoPath.1; .NET CLR 1.1.4322)

IE 8:
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; InfoPath.1; .NET CLR 1.1.4322)

Firefox:
Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9.0.5)Gecko/2008120122 Firefox/3.0.5
Mozilla/5.0 (X11; U; Linux i686; zh-CN; rv:1.9.0.4)Gecko/2008111318 Ubuntu/8.10 (intrepid)Firefox/3.0.4

Safari:
Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN)AppleWebKit/525.19 (KHTML, like Gecko)Version/3.1.2 Safari/525.21

Chrome:
Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US)AppleWebKit/525.19 (KHTML, like Gecko)Chrome/1.0.154.36 Safari/525.19

Opera:
Opera/9.63 (Windows NT 5.1; U; zh-cn)Presto/2.1.1
Opera/9.63 (X11; Linux i686; U; zh-cn)Presto/2.1.1
Opera/9.64 (Windows NT 5.1; U; en)Presto/2.1.1

Konqueror:
Mozilla/5.0 (compatible; Konqueror/4.1; Linux)KHTML/4.1.2 (like Gecko)
```

看来上面的一些数据后，或许你便明白了User-Agent的“书写公式”了。

`公司/版本(系统标志;语言)软件版本等描述说明`

你或许会发问，知道这些垃圾信息有什么用，我们又不是HTTP Server，没用啊。

你错了，这个信息是很有用的，我们可以通过总结归纳来仿造一些标识或者进行标识的样本库归纳，

并编写一些Server Application来达到保护我们的资源等目的。

