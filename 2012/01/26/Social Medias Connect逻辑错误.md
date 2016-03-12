# Social Medias Connect逻辑错误

用着Social Medias Connect感觉挺好的,但是今天检查评论的时候发现有一个头像处理有问题.

截取部分代码,主要输出的时候就是这段,经过查看那个用户是通过微博注册的,

但是因为要接受评论回复内容,就修改了邮箱.然后储存在数据库的邮箱内容就变化了,但是smcdata的标识却没有变化,

于是就出现了用户头像返回`src='/50'`之类的错误了,

```php
<?php
if($user_id && ($smcdata = get_usermeta($user_id, 'smcdata'))){
	if($smcdata['smcid']){
		$weibo=$smcdata['smcweibo'];
			switch($weibo){
				case 'sinaweibo':
					$out = 'http://tp3.sinaimg.cn/'.$smcdata['smcid'].'/50/1.jpg';
					break;
				case 'qqweibo':
						$out = 'http://app.qlogo.cn/mbloghead/'.$smcdata['smcid'].'/100';
						break;
				case 'douban':
						$out = 'http://img3.douban.com/icon/u'.$smcdata['smcid'].'.jpg';
						break;
				case 'sohuweibo':
						$out = $smcdata['smcid'];
						break;
				case '163weibo':
						$out = $smcdata['smcid'];
						break;
				default:return $avatar;
			}
		}else $out=$smcdata['avatar'];
		if(!$out)return $avatar;

		$avatar = "<img alt='' src='{$out}' class='avatar avatar-{$size}' height='{$size}' width='{$size}' />";

		return $avatar;
?>
```

修改方法也很简单,因为是由于修改邮箱后,程序依然进行字串处理才发生的错误,

所以只要让程序不继续进行下去就可以了,在第一个判断之下,添加一个小判断`if (strpos($avatar,'gravatar.com')){return $avatar;}`,然后就妥了。

```php
<?php
if($user_id && ($smcdata = get_usermeta($user_id, 'smcdata'))){
	if (strpos($avatar,'gravatar.com')){return $avatar;}
	if($smcdata['smcid']){
		$weibo=$smcdata['smcweibo'];
			switch($weibo){
				case 'sinaweibo':
					$out = 'http://tp3.sinaimg.cn/'.$smcdata['smcid'].'/50/1.jpg';
					break;
				case 'qqweibo':
						$out = 'http://app.qlogo.cn/mbloghead/'.$smcdata['smcid'].'/100';
						break;
				case 'douban':
						$out = 'http://img3.douban.com/icon/u'.$smcdata['smcid'].'.jpg';
						break;
				case 'sohuweibo':
						$out = $smcdata['smcid'];
						break;
				case '163weibo':
						$out = $smcdata['smcid'];
						break;
				default:return $avatar;
			}
		}else $out=$smcdata['avatar'];
		if(!$out)return $avatar;

		$avatar = "<img alt='' src='{$out}' class='avatar avatar-{$size}' height='{$size}' width='{$size}' />";

		return $avatar;
?>
```

[![dragon](https://attachment.soulteary.com/2012/01/26/dragon.png "dragon")](https://attachment.soulteary.com/2012/01/26/dragon.png)

