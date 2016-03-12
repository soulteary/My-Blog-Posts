# FirePhp Chrome版

> 如果你对这个插件有任何建议或者意见，欢迎在这里或者微博或者GitHub上留言。

## 写在前面

FirePhp是一款优秀的的PHP调试工具，拥有众多的粉丝。@luofei614童鞋不经意的一句提醒，发现这个插件在chrome和firefox下的战斗力根本就不是一个等级。然后，就有了这个插件。

## 相关资源

- 本插件GitHub: https://github.com/soulteary/firephp-for-chrome-sae
- FIREPHP插件官方: http://www.firephp.org/
- FIREPHP插件Google Code: http://code.google.com/p/firephp/

## 简单说明

使用这个插件的一般都是具有一定开发经验的Phper，所以傻瓜化的教程，我觉得大可不必。

不过插件还是使用很简单的，打开你的chrome的插件页面，勾选开发者模式，把插件拖动到chrome中，会提示安装。 如图所示。

[![2012-11-27_193128](https://attachment.soulteary.com/2012/11/28/2012-11-27_193128.png "2012-11-27_193128")](https://attachment.soulteary.com/2012/11/28/2012-11-27_193128.png) 

安装好之后，点击浏览器上的小鸡，开启插件调试即可。

[![20121127193447](https://attachment.soulteary.com/2012/11/28/20121127193447.png "20121127193447")](https://attachment.soulteary.com/2012/11/28/20121127193447.png)

## 测试代码

```php

'val1',
         'key2'=>array(array('v1','v2'),'v3')),
   'TestArray',FirePHP::LOG);

function test($Arg1) {
  throw new Exception('Test Exception');
}
try {
  test(array('Hello'=>'World'));
} catch(Exception $e) {
  /* Log exception including stack trace & variables */
  fb($e);
}

fb(array('2 SQL queries took 0.06 seconds',array(
   array('SQL Statement','Time','Result'),
   array('SELECT * FROM Foo','0.02',array('row1','row2')),
   array('SELECT * FROM Bar','0.04',array('row1','row2'))
  )),FirePHP::TABLE);

?>
```

## 已知BUG

1.由于CHROME限制HEADER信息数据大小,最大长度在500~600KB,FIREFOX无此限制(2MB的文件都妥妥的...) 所以调试的时候貌似不能输出太多的内容。issue @luofei614

## 计划features

1. Warn和Error弹出一个桌面提示,点击切换到提示内容区域。idea @luofei614 
2. 输出结构体用单独的小控件来做 idea @soulteary 
3. 响应更灵敏 && 数据支持过滤显示 idea @soulteary
4. 和另外一位chrome fans @忘汐 讨论,打算把插件展示改为dev_tools中展示
5. 和@luofei614同学讨论,打算用cache绕过header大小限制。  

## 界面预览

- URL参数显示 [![2012-11-27_181940](https://attachment.soulteary.com/2012/11/28/2012-11-27_181940.png "2012-11-27_181940")](https://attachment.soulteary.com/2012/11/28/2012-11-27_181940.png) 
- 表格显示 [![2012-11-27_182057](https://attachment.soulteary.com/2012/11/28/2012-11-27_182057.png "2012-11-27_182057")](https://attachment.soulteary.com/2012/11/28/2012-11-27_182057.png) 
- LABEL显示 [![2012-11-27_182046](https://attachment.soulteary.com/2012/11/28/2012-11-27_182046.png "2012-11-27_182046")](https://attachment.soulteary.com/2012/11/28/2012-11-27_182046.png) 
- 数组结构展示 [![2012-11-27_182008](https://attachment.soulteary.com/2012/11/28/2012-11-27_182008.png "2012-11-27_182008")](https://attachment.soulteary.com/2012/11/28/2012-11-27_182008.png)

