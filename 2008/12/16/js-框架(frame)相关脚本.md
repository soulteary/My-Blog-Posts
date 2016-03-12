# [js]框架(frame)相关脚本

页面中使用框架

```js
<script language="JavaScript">
<!--
if (window == top)top.location.href = "frames.htm"; //frames.htm为框架网页
//-->
</script>
```

防止被人加入框架


```js
<script language="JavaScript">
<!--
if (top.location != self.location)top.location=self.location;
//-->
</script>
```

