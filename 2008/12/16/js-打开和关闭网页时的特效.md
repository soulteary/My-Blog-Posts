# [js]打开和关闭网页时的特效

打开网页时的特效

```html
<meta http-equiv="Page-Enter" content="revealTrans(duration=x, transition=y)">
```


关闭网页时的特效

```html
<meta http-equiv="Page-Exit" content="revealTrans(duration=x, transition=y)">
```

## 说明

- duration表示特效的持续时间，单位为秒。
- transition表示使用的特效效果,可以选择0-23：

```text
0 矩形缩小
1 矩形扩大
2 圆形缩小
3 圆形扩大
4 下到上刷新
5 上到下刷新
6 左到右刷新
7 右到左刷新
8 竖百叶窗
9 横百叶窗
10 错位横百叶窗
11 错位竖百叶窗
12 点扩散
13 左右到中间刷新
14 中间到左右刷新
15 中间到上下
16 上下到中间
17 右下到左上
18 右上到左下
19 左上到右下
20 左下到右上
21 横条
22 竖条
23 以上22种随机选择一种
```

