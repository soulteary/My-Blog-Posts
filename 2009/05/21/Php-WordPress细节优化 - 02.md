# [Php]WordPress细节优化 - 02

书接上回，打算把iNove全部的拆开心得写上，直接的代码我是不会展示的，因为授人鱼不如授之渔。 这回送死的是 comments.php【小声嘀咕，我运气真背...】 

<!-- more -->

或许你和我同样第一眼看到的一个主过程便是它了。


```php
get_results($wpdb->prepare("SELECT * FROM $wpdb->comments WHERE comment_post_ID = %d AND comment_approved = '1' AND (comment_type = 'pingback' OR comment_type = 'trackback') ORDER BY comment_date", $post->ID));
 }
?>
```

这里MG为了通吃wordpress，或者说照顾有念旧情节的朋友，做了一个判断， 我的WordPress 2.7 or higher，所以我就直接把这种判断删除了，留言肯定会执行的就好，这么一来，便少了编译器的一次判断，一次不明显，如果嵌套10次以上的话，当然只是举个例子。修改后的或许会变成这个样子。这种兼容性的过程还有很多，包括一些插件以及旧的API函数中哦。

```php
_e('Comments', 'inove'); echo (' (' . (count($comments)-count($trackbacks)) . ')');  >
  [ _e('Trackbacks', 'inove'); echo (' (' . count($trackbacks) . ')');  >](javascript:void(0);)
?>
```

又看到这种兼容性很高的国际化语句了，俺们要的是速度咯，改。

```php
评论数 ( echo ((count($comments)-count($trackbacks)));  >) 
  [引用数 ( echo (count($trackbacks));  >)](javascript:void(0);) 

?>
```

当然类似的还有很多，参考上面的方法你的显示速度应该会有所提高。在参照上面的方法修缮了许多代码之后，我们又看到了一个这样的过程

```php

<div class="date">
        printf( __('%1$s at %2$s', 'inove'), get_comment_time(__('F jS, Y', 'inove')), get_comment_time(__('H:i', 'inove')) );         | [ printf('#%1$s', ++$trackbackcount);  >](#comment- comment_ID()  >)
      </div>

?>
```

这个也是多个函数嵌套的实现方法，某些时候，结构越是简单，效率越高。 于是改为

```php

<div class="date">
        echo get_comment_time('F jS, Y 于 H:i');  > | [ printf('#%1$s', ++$trackbackcount);  >](#comment- comment_ID()  >)
      </div>

?>
```

接着向下看，又看到了一个熟悉的东西，还是复习一下吧

```php
logged in to post a comment.', 'inove'), $login_link); 
?>
```

```php
">点此注册。

?>
```

是不是直接脱去了两层函数呢，一层是语言包，一层是变量合并。 继续往下看：很多人想搞一个和MG评论回复一样的效果，看看本文末端的评论表情就知道是啥了。 但是却不乏失败的...主要原因是因为你没有开启转换表情，导致函数数组为空，从而无法获取表情。 

详细的可以看[http://promiseforever.com/2009/05/11/wordpresshighslide4wp-hotfix.html](http://promiseforever.com/2009/05/11/wordpresshighslide4wp-hotfix.html) 

这篇是我写的<span style="color: #2970a6;">WordPress]highslide4wp 插件补充，</span>

```php
 highslide_emoticons();  >
    endif; 
?>
```

已经和MG反馈了，MG说这个是一个BUG... 如果你不启用这个插件的话，那么可以直接把上面的语句删除。 如果你怕你哪天不小心把表情转换点没了，造成程序bug，那么修改成下面的样子。 如果你不确定的话，那么也这样修改一下吧。

```php
 `if ((function_exists('highslide_emoticons')) && get_option( 'use_smilies' )):     

<div id="emoticon"> highslide_emoticons();  ></div>

    endif;  >`

?>
```

待续....

