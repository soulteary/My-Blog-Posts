# [Php]博客收藏以及转帖地址显示插件(5.27更新)

**本文于5月27日更新** 

感谢bolo wulinfo 的提醒，如果还有bug欢迎反馈。

最近太忙了,实在是不好意思,晚些时间我会一一回访每位来客的，感谢支持与理解。记得有朋友和我要文章底下的搜藏和地址显示插件。现在我把插件直接贴出来，大家将就的用吧~图片基本是我P过的，如果觉得不符合您的美学请自行替换。

还是之前的原则，授人鱼不如授人以渔。我先贴出如何修改，如果有人有疑问的话，我会贴出附件。:)

<!-- more -->

首先建立一个文本文件，然后把这个文件里加入以下代码。最后把我的css文件以及图片按钮保存起来就可以使用了。

欢迎反馈改进的建议以及意见。

```php
<?php
/*
**
*Plugin Name: 转帖&收藏插件
*Plugin URI: http://www.promiseforever.com/
*Description: 文章转载推荐以及地址快速复制。
*Author: firendless
*Version: 1.1
*Author URI: http://www.promiseforever.com/
*License: GNU General Public License 2.0 http://www.gnu.org/licenses/gpl.html
**/

//添加HOOK
add_filter('the_content', 'theshareContentFilter');

//添加收藏盒子以及文章地址
function firendlessShareBox($mustEcho = true) {

//只在文章页面添加书签盒子以及本文地址
if (is_single()) {
$theHTMLCode = '';
//获取文章地址
$strLinkTmps = get_permalink();
//获取文章标题
$title = the_title('','',false);
$theHTMLCode = '<p clsss="postmet"><a href="'.$strLinkTmps.' title="本文地址[转载请说明，谢谢合作。]">本文地址[转载请说明，谢谢合作。]</a><br /><textarea id="firlink">'.$strLinkTmps.'</textarea></p>';
//获取文章地址并转换为类似JS转码过的地址
$url = urlencode(get_permalink());
//进行书签处理"=?UTF-8 b?".base64_encode('邮件标题')."?=";
$theHTMLCode.= __('<div id="ShareButton"><strong>觉得本文不错？那么用下面的方式分享本文吧[推荐/书签]</strong><ul>
<li id="shareMail"><a rel="external nofollow" class="sharebtn" title="直接发送邮件给朋友。" href="mailto:?subject=' .$title. '&amp;body=' . $strLinkTmps . '" target="_blank"><span>Mail To</span></a></li>
<li id="sharefacebook"><a rel="external nofollow" class="sharebtn" title="使用FaceBook分享或收藏此文。" href="http://www.facebook.com/share.php?u=' . $url . '" target="_blank"><span>facebook</span></a></li>
<li id="sharetwitter"><a rel="external nofollow" class="sharebtn" title="使用Twitter分享或收藏此文。" href="http://twitter.com/home?status=' . $url . '" target="_blank"><span>twitter</span></a></li>
<li id="sharedigg"><a rel="external nofollow" class="sharebtn" title="使用digg分享或收藏此文。" href="http://digg.com/submit?url=' . $url . '&amp;title=' . $title . '&amp;bodytext=&amp;media=&amp;topic=" target="_blank"><span>digg</span></a></li>
<li id="sharedelicious"><a rel="external nofollow" class="sharebtn" title="使用delicious分享或收藏此文。" href="http://delicious.com/save?v=5&amp;noui&amp;jump=close&amp;url=' . $url . '&amp;title=' . $title .'" target="_blank"><span>delicious</span></a></li>
<li id="sharebaidu"><a rel="external nofollow" class="sharebtn" title="使用百度分享或收藏此文。" href="http://cang.baidu.com/do/add?it=' . $title . '&iu=' . $url . '&fr=ien#nw=1" target="_blank"><span>Baidu</span></a></li>
<li id="shareGoogle"><a rel="external nofollow" class="sharebtn" title="使用谷歌分享或收藏此文。" href="http://www.google.com/bookmarks/mark?op=edit&bkmk=' . $url . '&title=' . $title . '" target="_blank"><span>Google</span></a></li>
<li id="sharefanfou"><a rel="external nofollow" class="sharebtn" title="使用饭否分享或收藏此文。" href="http://fanfou.com/sharer?u=' . $url . '&t=' . $title . '" target="_blank"><span>饭否</span></a></li>
<li id="shareyahoo"><a rel="external nofollow" class="sharebtn" title="使用雅虎分享或收藏此文。" href="http://myweb2.search.yahoo.com/myresults/bookmarklet?u=' . $url . '&=' . $title . '' . $url . '&title=' . $title . '" target="_blank"><span>Yahoo</span></a></li>
<li id="shareQQ"><a rel="external nofollow" class="sharebtn" title="使用QQ书签分享或收藏此文。" href="http://shuqian.qq.com/post?from=3&title=' . $title . '&uri=' . $url . '&jumpback=2&noui=1" target="_blank"><span>QQ书签</span></a></li>
<li id="shareLive"><a rel="external nofollow" class="sharebtn" title="使用Live分享或收藏此文。" href="https://favorites.live.com/quickadd.aspx?marklet=1&url=' . $url . '&&title=' . $title . '" target="_blank"><span>Live</span></a></li>
<li id="sharexianguo"><a rel="external nofollow" class="sharebtn" title="使用鲜果分享或收藏此文。" href="http://xianguo.com/service/submitfav?link=' . $url . '&title=' . $title . '" target="_blank"><span>鲜果</span></a></li>
<li id="sharedouban"><a rel="external nofollow" class="sharebtn" title="使用豆瓣分享或收藏此文。" href="http://9.douban.com/recommend/?url=' . $url . '&title=' . $title . '" target="_blank"><span>豆瓣</span></a></li>
<li id="sharereddit"><a rel="external nofollow" class="sharebtn" title="使用reddit分享或收藏此文。" href="http://reddit.com/submit?url=' . $url . '&title=' . $title . '" target="_blank"><span>reddit</span></a></li>
<li id="sharemyspace"><a rel="external nofollow" class="sharebtn" title="使用myspace分享或收藏此文。" href="http://www.myspace.com/Modules/PostTo/Pages/?u=' . $url . '&t=' . $title . '" target="_blank"><span>myspace</span></a></li></ul></div><br />');
if($mustEcho) {
echo ($theHTMLCode);
} else {
return $theHTMLCode;
}
}
}

function theshareContentFilter($content = '')
{
if (!is_feed() && is_single()) {
return $content . firendlessShareBox(false);
} else {
return $content;
}
}
?>
```

接下来打开你所使用的主题的style.css添加以下内容

```css
 /* 分享按钮 */ #ShareButton ul {
 background:#eee;
 border:1px solid #B7B7B7;
 width:580px;
 padding:1px;
}
#ShareButton li {
 background:transparent;
 list-style:none;
 float:left;
 margin:0;
 padding:0;
 display:block;
} #ShareButton li a span {
 background:url(img/sharebutton.png) no-repeat;
 height:28px;
 width:24px;
 display:block;
 text-indent:-999em;
/* padding-top:10px; */
}
#ShareButton li#shareMail a span {
 background-position:-48px 0;
}
#ShareButton li#sharefacebook a span {
 background-position:-24px 0;
}
#ShareButton li#sharetwitter a span {
 background-position:-96px 0;
}
#ShareButton li#sharedigg a span {
 background-position:-72px 0;
}
#ShareButton li#sharedelicious a span {
 background-position:0 0;
}
#ShareButton li#sharebaidu a span {
 background-position:-144px 0;
}
#ShareButton li#shareGoogle a span {
 background-position:-168px 0;
}
#ShareButton li#sharefanfou a span {
 background-position:-120px 0;
}
#ShareButton li#shareQQ a span {
 background-position: -216px 0;
}
#ShareButton li#shareyahoo a span {
 background-position: -192px 0;
}
#ShareButton li#shareLive a span {
 background-position: -240px 0;
}
#ShareButton li#sharexianguo a span {
 background-position: -264px 0;
}
#ShareButton li#sharedouban a span {
 background-position: -336px 0;
}
#ShareButton li#sharemyspace a span {
 background-position: -312px 0;
}
#ShareButton li#sharereddit a span {
 background-position: -288px 0;
}
#ShareButton li a {
 display:block;
 padding:5px;
 text-decoration:none;
 width:24px;
 font-size:12px;
 background:#eee;
}
#ShareButton li a:hover {
 background:#fff;
} p.postmet a {font-size:12px;font-family:verdana;margin:5px 0}
#firlink a {border:none; color:#000;}
#firlink a:hover {color:#000}
textarea#firlink {width:430px;height:20px;padding:2px 0 0 2px; margin:5px}
```

接下来把图片放到你主题下的img/sharebutton.png 就大功告成了。

