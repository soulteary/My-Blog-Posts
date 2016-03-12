# Error-Tracer

[![2013-05-21_000000](https://attachment.soulteary.com/2013/05/21/2013-05-21_000000.png "2013-05-21_000000")](https://attachment.soulteary.com/2013/05/21/2013-05-21_000000.png)

Error Tracer 是一个简单的JS+PHP实现的前端错误追踪应用，可以说是为了FIX线上的脚本错误而尝试做的一个比较幼稚的东西。

这个想法几个月之前就有，实际落实发现不过2，3天时间，当然，程序尚有很多可以完善的地方。

DEMO版本的界面山寨了NODEJS的ErrorBoard， 现在跑在SINA APP ENGINE。

## 目前的特性：

1. 数据库主从分离。
2. 自动添加浏览器信息。
3. 自动合并和计算同类型的错误信息。
4. 自动计算BUG生命周期。

## 打算加入的features:

1. 浏览器信息支持列表+自动添加。
2. 合并和计算同类型信息更精确。
3. 完成修复的BUG直接在线删除或者标记。
4. 支持根据浏览器，IP，脚本，行数，来查找和追踪触发BUG的人群。
5. 汇总邮件。

- DEMO地址: http://errorboard.sinaapp.com/ 
- DEMO后台: http://errorboard.sinaapp.com/push/?mode=admin

