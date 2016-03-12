# [PHP]用数组帮WordPress确定输出特定分类的文章

当把博客或者说个人网站部分CMS化变的越来越流行的时候，输出特定的内容就显得特别重要。 

[![多项展示的分类内容](https://attachment.soulteary.com/2010/03/03/2010-03-03_140354.jpg "多项展示的分类内容")](https://attachment.soulteary.com/2010/03/03/2010-03-03_140354.jpg) 

譬如我的博客的热门分类的输出，如果你满足于输出某单项分类，仅仅需要在代码中添加下面的语句但是如果你想实现如文章附图中的效果，似乎就需要采取数组了。

```php
<?php
$cat_str=你想输出的ID内容;
query_posts("showposts=3&cat=".$cat_str);
while(have_posts()):the_post(); 
//这里输出你自己构造的内容
endwhile;
?>
```

但是如果你想实现如文章附图中的效果，似乎就需要采取数组了。

```
<?php
$cat_str = join(",",$你的分类数组);
query_posts("showposts=3&cat=".$cat_str);
while(have_posts()):the_post(); 
//这里输出你自己构造的内容
endwhile;
?>
```


