# Wordpress 3.X Gravatar修改方案

[![20120121154641](https://attachment.soulteary.com/2012/01/21/20120121154641.jpg "20120121154641")](https://attachment.soulteary.com/2012/01/21/20120121154641.jpg)

> 首先感谢 [willin](http://promiseforever.com/redirect?url=http://kan.willin.org/?p=1320&d=fir)，他是Gravatar的原创作者。

PHP 原始函数就不贴了，有兴趣的朋友可以自行去willin那里看下。

既然说到方案，那么如果是简单的修改一下就太噱头了。看完本文你可以实现的东西有:

1. 缓存Gravatar OPEN ID头像到你的网站的目录,以及完善的rewrite输出。
2. 游客使用邮箱评论的时候动态展示游客的Gravatar头像,增加交互.
3. 我可怜的小博客的浏览量+1...(可以无视)


首先实现缓存,free-domain cookies对于每个技术菜鸟来说都不陌生吧,小菜我表示如果静态资源不分离到子域名的话,浏览器加载的时候就不是很舒服..
所以我把Gravatar也缓存到了子目录...这个先按下不表,只是路径的问题.现在我说一下大众的简单修改方法.

首先在你的主题的function.php文件中追加函数和hook如下:
如果不会添加的话,可以参考willin的原文.链接在本文开头。

```php
function my_avatar($avatar) {
if (strpos($avatar,'gravatar.com')){

  $tmp = strpos($avatar, 'http');
  $g = substr($avatar, $tmp, strpos($avatar, "'", $tmp) - $tmp);
  $tmp = strpos($g, 'avatar/') + 7;
  $f = substr($g, $tmp, strpos($g, "?", $tmp) - $tmp);
  $w = WP_CONTENT_URL;//这里修改成你自己的位置
  $e = WP_CONTENT_DIR. '/avatar/'. $f. '.jpg';//这里修改成你自己的位置
  $t = 8035200; //設定3yue, 單位:秒
  if ( !is_file($e) || (time() - filemtime($e)) > $t ) { //當頭像不存在或文件超過14天才更新
    copy($g, $e);
  } else  {$avatar = strtr($avatar, array($g => $w.'/avatar/'.$f.'.jpg'));}
  if (filesize($e) < 500) {copy($w.'/avatar/default.jpg', $e);}
}
  return $avatar;
}
add_filter('get_avatar', 'my_avatar');
```

现在你的主题就能自动实现缓存了,第一次浏览的时候,图片的位置还是Gravatar服务器的,再次访问的话,就已经是自己的了.

接着,别太着急,这个方案也不是太完善哦,对于恶意的请求,和程序错误的请求,和其他的不正确的请求,我们也要做个判断.
直接在你的avatar目录中建立htaccess规则.其他服务器软件改写规则类似。

```apache
<FilesMatch "\.(jpg|jpeg|png|gif)$">
FileETag None
Header unset ETag
Header set Expires "Tue, 08 Dec 2020 20:02:54 GMT"
</FilesMatch>

<IfModule mod_deflate.c>
AddOutputFilter DEFLATE png gif jpg
</IfModule>

<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule .(.*) http:\/\/cache.promiseforever.com\/wp3\/avatar\/default.jpg [R,NC,L]
</IfModule>
```

当然你还可以加上判断refer的判断,杜绝简单的盗链..相信我.盗链还是用程序来做吧.至于小的静态资源,由它去吧..
这样设置之后,错误的访问就会转到你的默认图片上.而且图片们的浏览器缓存也设置完成,可以提高再次访问的速度。

接着是在评论的时候,添加动态展示了。
在[朋朋博客](http://promiseforever.com/redirect?url=http%3A%2F%2Fkan.willin.org%2F%3Fp%3D1320&key=a674a3c987dc8c52867cbc1ac3e897ab)看到这个功能后,觉得不错,但是没有添加有效性判断,于是修改之。

放一个简单的HTML结构

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>无标题文档</title>
</head>
<script type="text/javascript" src="jquery.js"></script>
<body>
<input type="text" name="email" id="email" class="commenttext" value="" size="22">
<div id="real-avatar">
<img alt="" src="http://cache.promiseforever.com/wp3/avatar/default.jpg" class="avatar photo avatar-default" height="60" width="60">
</div>
<a id="Get_Gravatar" title="请填写邮箱" target="_blank" href="http://promiseforever.com/contact" class="external">这个是你的头像么？</a>
</body>
</html>
```

然后是改进后的js代码，顺便到百度搜索了个在线的yui eval混淆.把后面的md5计算的包裹掉了.不喜欢包裹的可以去朋朋博客找原始的或者百度在线的解密eval.

```js
<script type="text/javascript">
jQuery(document).ready(function(){
	jQuery('#email').blur(function(){
	//验证邮箱地址有效性
	var fir_mail = jQuery('#email').val();
		if(!(fir_mail.match(/^\w+((-\w+)|(\.\w+))*\@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/))){
     		jQuery('#email1').focus();
	 		return fir_tips(1,fir_mail);
	 	}
	 //参与MD5计算 是大小写敏感的.
	 fir_mail=fir_mail.toLowerCase();
	//合法
	return fir_tips(2,fir_mail);
	});
});

function fir_tips($act,$mail){
	var $ret ='';
	if (!$act){return $ret='';}
	switch ($act)
	{
		case 1: $ret="邮箱格式不正确！";break;
		case 2: $ret="这个头像是你的嘛？";break;
　　		default: $ret="script by soulteary,is there anything ok?";
	}
	jQuery('#Get_Gravatar').fadeOut().html($ret).fadeIn('slow');
	jQuery('#real-avatar .avatar').attr('src','http://cache.promiseforever.com/wp3/avatar/' + hex_md5($mail)+'.jpg');
}
eval(function(p,a,c,k,e,d){e=function(c){return(c<a?'':e(parseInt(c/a)))+((c=c%a)>35?String.fromCharCode(c+29):c.toString(36))};if(!''.replace(/^/,String)){while(c--){d[e(c)]=k[c]||e(c)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('f P=0;f R="";l 1D(s){m 1l(W(z(s)))}l 1P(s){m Z(W(z(s)))}l 1V(s,e){m 1f(W(z(s)),e)}l 1F(k,d){m 1l(U(z(k),z(d)))}l 1E(k,d){m Z(U(z(k),z(d)))}l 1J(k,d,e){m 1f(U(z(k),z(d)),e)}l 1G(){m 1D("1I").1H()=="1Q"}l W(s){m 1d(N(Y(s),s.n*8))}l U(1g,1h){f I=Y(1g);C(I.n>16)I=N(I,1g.n*8);f 1j=D(16),1i=D(16);v(f i=0;i<16;i++){1j[i]=I[i]^1X;1i[i]=I[i]^25}f 1y=N(1j.1n(Y(1h)),1t+1h.n*8);m 1d(N(1i.1n(1y),1t+29))}l 1l(g){1x{P}1w(e){P=0}f 1e=P?"27":"2i";f h="";f x;v(f i=0;i<g.n;i++){x=g.w(i);h+=1e.Q((x>>>4)&1b)+1e.Q(x&1b)}m h}l Z(g){1x{R}1w(e){R=\'\'}f 1z="28+/";f h="";f A=g.n;v(f i=0;i<A;i+=3){f 1B=(g.w(i)<<16)|(i+1<A?g.w(i+1)<<8:0)|(i+2<A?g.w(i+2):0);v(f j=0;j<4;j++){C(i*8+j*6>g.n*8)h+=R;X h+=1z.Q((1B>>>6*(3-j))&E)}}m h}l 1f(g,T){f 1a=T.n;f i,j,q,x,K;f L=D(O.1C(g.n/2));v(i=0;i<L.n;i++){L[i]=(g.w(i*2)<<8)|g.w(i*2+1)}f 19=O.1C(g.n*8/(O.1A(T.n)/O.1A(2)));f S=D(19);v(j=0;j<19;j++){K=D();x=0;v(i=0;i<L.n;i++){x=(x<<16)+L[i];q=O.26(x/1a);x-=q*1a;C(K.n>0||q>0)K[K.n]=q}S[j]=x;L=K}f h="";v(i=S.n-1;i>=0;i--)h+=T.Q(S[i]);m h}l z(g){f h="";f i=-1;f x,y;1Y(++i<g.n){x=g.w(i);y=i+1<g.n?g.w(i+1):0;C(1Z<=x&&x<=24&&2b<=y&&y<=2c){x=2j+((x&1u)<<10)+(y&1u);i++}C(x<=2k)h+=F.G(x);X C(x<=2l)h+=F.G(2m|((x>>>6)&1W),H|(x&E));X C(x<=V)h+=F.G(2h|((x>>>12)&1b),H|((x>>>6)&E),H|(x&E));X C(x<=2d)h+=F.G(2e|((x>>>18)&2f),H|((x>>>12)&E),H|((x>>>6)&E),H|(x&E))}m h}l 2g(g){f h="";v(f i=0;i<g.n;i++)h+=F.G(g.w(i)&J,(g.w(i)>>>8)&J);m h}l 2n(g){f h="";v(f i=0;i<g.n;i++)h+=F.G((g.w(i)>>>8)&J,g.w(i)&J);m h}l Y(g){f h=D(g.n>>2);v(f i=0;i<h.n;i++)h[i]=0;v(f i=0;i<g.n*8;i+=8)h[i>>5]|=(g.w(i/8)&J)<<(i%32);m h}l 1d(g){f h="";v(f i=0;i<g.n*32;i+=8)h+=F.G((g[i>>5]>>>(i%32))&J);m h}l N(x,A){x[A>>5]|=H<<((A)%32);x[(((A+1R)>>>9)<<4)+14]=A;f a=1S;f b=-1T;f c=-1U;f d=1K;v(f i=0;i<x.n;i+=16){f 1p=a;f 1r=b;f 1o=c;f 1v=d;a=p(a,b,c,d,x[i+0],7,-1L);d=p(d,a,b,c,x[i+1],12,-1M);c=p(c,d,a,b,x[i+2],17,1N);b=p(b,c,d,a,x[i+3],22,-1O);a=p(a,b,c,d,x[i+4],7,-2a);d=p(d,a,b,c,x[i+5],12,2D);c=p(c,d,a,b,x[i+6],17,-35);b=p(b,c,d,a,x[i+7],22,-34);a=p(a,b,c,d,x[i+8],7,36);d=p(d,a,b,c,x[i+9],12,-37);c=p(c,d,a,b,x[i+10],17,-38);b=p(b,c,d,a,x[i+11],22,-33);a=p(a,b,c,d,x[i+12],7,31);d=p(d,a,b,c,x[i+13],12,-2o);c=p(c,d,a,b,x[i+14],17,-2W);b=p(b,c,d,a,x[i+15],22,2V);a=o(a,b,c,d,x[i+1],5,-2X);d=o(d,a,b,c,x[i+6],9,-3a);c=o(c,d,a,b,x[i+11],14,30);b=o(b,c,d,a,x[i+0],20,-2Z);a=o(a,b,c,d,x[i+5],5,-39);d=o(d,a,b,c,x[i+10],9,3c);c=o(c,d,a,b,x[i+15],14,-3l);b=o(b,c,d,a,x[i+4],20,-3k);a=o(a,b,c,d,x[i+9],5,3i);d=o(d,a,b,c,x[i+14],9,-3d);c=o(c,d,a,b,x[i+3],14,-3j);b=o(b,c,d,a,x[i+8],20,3b);a=o(a,b,c,d,x[i+13],5,-3e);d=o(d,a,b,c,x[i+2],9,-3f);c=o(c,d,a,b,x[i+7],14,3h);b=o(b,c,d,a,x[i+12],20,-3g);a=u(a,b,c,d,x[i+5],4,-2Y);d=u(d,a,b,c,x[i+8],11,-2T);c=u(c,d,a,b,x[i+11],16,2y);b=u(b,c,d,a,x[i+14],23,-2x);a=u(a,b,c,d,x[i+1],4,-2z);d=u(d,a,b,c,x[i+4],11,2A);c=u(c,d,a,b,x[i+7],16,-2C);b=u(b,c,d,a,x[i+10],23,-2U);a=u(a,b,c,d,x[i+13],4,2w);d=u(d,a,b,c,x[i+0],11,-2v);c=u(c,d,a,b,x[i+3],16,-2q);b=u(b,c,d,a,x[i+6],23,2p);a=u(a,b,c,d,x[i+9],4,-2r);d=u(d,a,b,c,x[i+12],11,-2s);c=u(c,d,a,b,x[i+15],16,2u);b=u(b,c,d,a,x[i+2],23,-2t);a=r(a,b,c,d,x[i+0],6,-2E);d=r(d,a,b,c,x[i+7],10,2O);c=r(c,d,a,b,x[i+14],15,-2N);b=r(b,c,d,a,x[i+5],21,-2P);a=r(a,b,c,d,x[i+12],6,2Q);d=r(d,a,b,c,x[i+3],10,-2S);c=r(c,d,a,b,x[i+10],15,-2R);b=r(b,c,d,a,x[i+1],21,-2L);a=r(a,b,c,d,x[i+8],6,2G);d=r(d,a,b,c,x[i+15],10,-2F);c=r(c,d,a,b,x[i+6],15,-2H);b=r(b,c,d,a,x[i+13],21,2I);a=r(a,b,c,d,x[i+4],6,-2J);d=r(d,a,b,c,x[i+11],10,-2B);c=r(c,d,a,b,x[i+2],15,2K);b=r(b,c,d,a,x[i+9],21,-2M);a=B(a,1p);b=B(b,1r);c=B(c,1o);d=B(d,1v)}m D(a,b,c,d)}l M(q,a,b,x,s,t){m B(1q(B(B(a,q),B(x,t)),s),b)}l p(a,b,c,d,x,s,t){m M((b&c)|((~b)&d),a,b,x,s,t)}l o(a,b,c,d,x,s,t){m M((b&d)|(c&(~d)),a,b,x,s,t)}l u(a,b,c,d,x,s,t){m M(b^c^d,a,b,x,s,t)}l r(a,b,c,d,x,s,t){m M(c^(b|(~d)),a,b,x,s,t)}l B(x,y){f 1k=(x&V)+(y&V);f 1s=(x>>16)+(y>>16)+(1k>>16);m(1s<<16)|(1k&V)}l 1q(1m,1c){m(1m<<1c)|(1m>>>(32-1c))}',62,208,'|||||||||||||||var|input|output||||function|return|length|md5_gg|md5_ff||md5_ii|||md5_hh|for|charCodeAt|||str2rstr_utf8|len|safe_add|if|Array|0x3F|String|fromCharCode|0x80|bkey|0xFF|quotient|dividend|md5_cmn|binl_md5|Math|hexcase|charAt|b64pad|remainders|encoding|rstr_hmac_md5|0xFFFF|rstr_md5|else|rstr2binl|rstr2b64||||||||||full_length|divisor|0x0F|cnt|binl2rstr|hex_tab|rstr2any|key|data|opad|ipad|lsw|rstr2hex|num|concat|oldc|olda|bit_rol|oldb|msw|512|0x03FF|oldd|catch|try|hash|tab|log|triplet|ceil|hex_md5|b64_hmac_md5|hex_hmac_md5|md5_vm_test|toLowerCase|abc|any_hmac_md5|271733878|680876936|389564586|606105819|1044525330|b64_md5|900150983cd24fb0d6963f7d28e17f72|64|1732584193|271733879|1732584194|any_md5|0x1F|0x36363636|while|0xD800|||||0xDBFF|0x5C5C5C5C|floor|0123456789ABCDEF|ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789|128|176418897|0xDC00|0xDFFF|0x1FFFFF|0xF0|0x07|str2rstr_utf16le|0xE0|0123456789abcdef|0x10000|0x7F|0x7FF|0xC0|str2rstr_utf16be|40341101|76029189|722521979|640364487|421815835|995338651|530742520|358537222|681279174|35309556|1839030562|1530992060|1272893353|1120210379|155497632|1200080426|198630844|30611744|1873313359|1560198380|1309151649|145523070|718787259|2054922799|343485551|1416354905|1126891415|57434055|1700485571|1051523|1894986606|2022574463|1094730640|1236535329|1502002290|165796510|378558|373897302|643717713|1804603682||1990404162|45705983|1473231341|1770035416|1958414417|42063|701558691|1069501632|1163531501|38016083|1019803690|1444681467|51403784|1926607734|1735328473|568446438|187363961|405537848|660478335'.split('|'),0,{}))
</script>
```

至此，方案讲述完毕。

