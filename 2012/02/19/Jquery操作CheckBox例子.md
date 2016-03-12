# jQuery操作CheckBox例子

一个不错的例子,包含了jQuery操作CheckBox的例子。

<!-- more -->

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
<title>无标题页</title>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.2.6/jquery.min.js" type="text/javascript"></script>
<script type="text/javascript">
jQuery(function($){
//全选
$("#btn1").click(function(){
$("input[name='checkbox']").attr("checked","true");
})
//取消全选
$("#btn2").click(function(){
$("input[name='checkbox']").removeAttr("checked");
})
//选中所有基数
$("#btn3").click(function(){
$("input[name='checkbox']:even").attr("checked","true");
})
//选中所有偶数
$("#btn6").click(function(){
$("input[name='checkbox']:odd").attr("checked","true");
})
//反选
$("#btn4").click(function(){
$("input[name='checkbox']").each(function(){
if($(this).attr("checked"))
{
$(this).removeAttr("checked");
}
else
{
$(this).attr("checked","true");
}
})
})



//或许选择项的值
var aa="";
$("#btn5").click(function(){

$("input[type=checkbox][checked]").each(function(){ //由于复选框一般选中的是多个,所以可以循环输出
alert($(this).val());
});

})
})
</script>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>
<body>
<form id="form1" runat="server">
  <div>
    <input type="button" id="btn1" value="全选">
    <input type="button" id="btn2" value="取消全选">
    <input type="button" id="btn3" value="选中所有奇数">
    <input type="button" id="btn6" value="选中所有偶数">
    <input type="button" id="btn4" value="反选">
    <input type="button" id="btn5" value="获得选中的所有值">
    <br>
    <input type="checkbox" name="checkbox" value="checkbox1">
    checkbox1
    <input type="checkbox" name="checkbox" value="checkbox2">
    checkbox2
    <input type="checkbox" name="checkbox" value="checkbox3">
    checkbox3
    <input type="checkbox" name="checkbox" value="checkbox4">
    checkbox4
    <input type="checkbox" name="checkbox" value="checkbox5">
    checkbox5
    <input type="checkbox" name="checkbox" value="checkbox6">
    checkbox6
    <input type="checkbox" name="checkbox" value="checkbox7">
    checkbox7
    <input type="checkbox" name="checkbox" value="checkbox8">
    checkbox8 </div>
</form>
</body>
</html>

```

