# [js]如何防止网页中的内容被复制

防止网页中的内容被复制,禁止复制网页内容

取消选取、防止复制

```html
<body onselectstart="return false">
```

方法二
```html
<body oncopy="return false;" oncut="return false;">
```

