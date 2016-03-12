# [PHP]Discuz! X2 预防灌水修改

[![2011-07-20_070211](https://attachment.soulteary.com/2011/07/2011-07-20_070211.png "2011-07-20_070211")](https://attachment.soulteary.com/2011/07/2011-07-20_070211.png) 

这两天突然回忆起以前上学校论坛了，于是连续两天半夜都上去浏览。
发现一个事情，似乎有人在用灌水机器发不良信息。而且时间持续了很久了。
暑假假期是招生时期，这个时候，从论坛传出的一些信息会对学校产生负面的影响。
所以写了一个小功能，来防止机械工具的灌水。

只是一个DEMO，希望论坛的管理能重视并修改。
这个DEMO，应该能预防一般的发帖机器，如果哪个发帖的人继续升级软件的话。
那么我们也跟进算法，修改功能就可以避免不良信息了。

<!-- more -->

首先是修改\source\module\misc\目录下的misc_seccode.php
搜索以下语句

```php
<?php
$message = lang('core', 'seccode_image'.$ani.'_tips').'<img onclick="updateseccode(\''.$idhash.'\')" width="'.$_G['setting']['seccodedata']['width'].'" height="'.$_G['setting']['seccodedata']['height'].'" src="misc.php?mod=seccode&update='.$rand.'&idhash='.$idhash.'" class="vm" alt="" />';
?>
```

在其下面添加

```php
<?php
$fir_mask= 'theKey';
$fir_name= md5($idhash.$fir_mask);
$message.= '<br /><input type="checkbox" name="'.$fir_name.'" /><label for="'.$fir_name.'">我保证不发布不良内容</label>';
?>
```

这样，验证图片下面就多了一个单选框，“我保证不发布不良内容”

[![2011-07-20_070211](https://attachment.soulteary.com/2011/07/2011-07-20_062502.png "2011-07-20_070211")](https://attachment.soulteary.com/2011/07/2011-07-20_062502.png) 

然后修改\source\function目录下的function_core.php
首先搜索

```php
<?php
function check_seccode($value, $idhash) {
	global $_G;
	if(!$_G['setting']['seccodestatus']) {
		return true;
	}
	if(!isset($_G['cookie']['seccode'.$idhash])) {
		return false;
	}
?>
```

在这里添加

```php
<?php
	$fir_mask= 'theKey';
	$fir_name= md5($idhash.$fir_mask);
	$ret = $_GET[$fir_name] =='on';
	if (false == $ret){return false;}

?>
```

这样，就完成了，在验证码后出现勾选同意与否的CHECKBOX，对于用户来说，那个勾选是很轻松的，对于灌水机器来说，就需要升级才能支持，如果他继续升级软件进行不良内容发布的话，那么，说明他是有意攻击论坛，那么我们将上面的简单CHECKBOX替换为一个随机的逻辑验证就能轻松解决了。

但是到这里，程序还是没有完毕，因为你会发现，无论你验证码填写是否正确，AJAX的验证码填写正确/错误，一直显示都是错误，虽然不影响用户，但是影响美观。

继续修改有两种方法，其1，直接删除此检查函数。
\static\js 目录内common.js
搜索并删除此函数

```js
function checksec(type, idhash, showmsg, recall) {
	$F('_checksec', arguments);
}
```

\static\js 目录内common_extra.js
搜索并删除此函数

```js
function _checksec(type, idhash, showmsg, recall) {
	var showmsg = !showmsg ? 0 : showmsg;
	var secverify = $('sec' + type + 'verify_' + idhash).value;
	if(!secverify) {
		return;
	}
	var x = new Ajax('XML', 'checksec' + type + 'verify_' + idhash);
	x.loading = '';
	$('checksec' + type + 'verify_' + idhash).innerHTML = '<img src="'+ IMGDIR + '/loading.gif" width="16" height="16" class="vm" />';
	x.get('misc.php?mod=sec' + type + '&action=check&inajax=1&&idhash=' + idhash + '&secverify=' + (BROWSER.ie && document.charset == 'utf-8' ? encodeURIComponent(secverify) : secverify), function(s){
		var obj = $('checksec' + type + 'verify_' + idhash);
		obj.style.display = '';
		if(s.substr(0, 7) == 'succeed') {
			obj.innerHTML = '<img src="'+ IMGDIR + '/check_right.gif" width="16" height="16" class="vm" />';
			if(showmsg) {
				recall(1);
			}
		} else {
			obj.innerHTML = '<img src="'+ IMGDIR + '/check_error.gif" width="16" height="16" class="vm" />';
			if(showmsg) {
				if(type == 'code') {
					showError('验证码错误，请重新填写');
				} else if(type == 'qaa') {
					showError('验证问答错误，请重新填写');
				}
				recall(0);
			}
		}
	});
}
```

这个方案的好处是少了一次服务器资源使用，坏处就是不显示了验证提示。
如果觉得用户体验还是要兼顾，那么我们使用方案2.
\static\js 目录内common_extra.js
首先说一下，较完美的方法是这样：
将方案1中准备使用的函数进行修改，
加上之前php中添加的内容，当然，之前的input 还要加上一个ID。
这种方法把input的id写死，个人感觉不妥。补充一下，MD5类,DZ有带，使用方法hex_md5(str)。
我觉得这个方法，比较容易被突破，所以，修改提示好了，让提示模糊处理用户操作。
当然，如果PHP中的验证CHECKBOX验证,单独写开,可以不修改JS,坏处就是你要修改所有调用check_seccode函数的文件了。
如果使用了插件，或者未来使用的插件会调用这个函数，就会出现防护覆盖不到的问题。


还是那个函数，将里面的提示图片的语句删除，找到下面2句语句并删除即可。

```js
obj.innerHTML = '<img src="'+ IMGDIR + '/check_right.gif" width="16" height="16" class="vm" />';
```

```js
obj.innerHTML = '<img src="'+ IMGDIR + '/check_error.gif" width="16" height="16" class="vm" />';
```

这样输出错验证码，依旧会提示消息。你还可以把JS错误提示的那段中文修改为，“需要同意发帖协议以及输出正确验证码”

就像前面说的一样，这只是一个DEMO,修改2个PHP文件,一处是显示调用,一处是函数处理.
修改2个JS,一处是调用,一处是实现.

如果不想修改JS,那么就搜索所有的调用check_seccode函数，在每个他的调用下面添加一个自定义函数，内容就是检查CHECK_BOX.

如果你想直接使用这些代码的话，为了安全性，请修改

```php
<?php
	$fir_mask= 'theKey';
?>
```

如果有其他的问题，欢迎留言。

