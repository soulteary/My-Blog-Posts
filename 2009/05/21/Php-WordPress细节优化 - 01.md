# [Php]WordPress细节优化 - 01

现在随便逛几个博客就能看到MG12做的iNove，呵呵，我这个也是，经过一个礼拜的解剖，俺打算把解剖心得发上来留个纪念。[bt...啥心理，解剖了人家的东西还要说心得。]

个人觉得wordpress fans们最关注的莫过于wordpress的速度了。

所以先从Php脚本开始说吧，总所周知一个好的计划会避免走很多岔路从而提高工作学习的效率，程序也是如此。

不过在程序中，这个名词被换成了“程序结构”，有良好的结构的话，可以减少不必要的损耗，从而提高反应速度。

如果我们同样使用这款iNove，在插件和服务器环境一样，流量相同等情况下，【控制变量法】

假设我的程序要请求10次数据库，你的要请求20次，我的就会比你的快一点点。

积少成多，我们的速度便会明显起来了。

<!-- more -->

比如我们打开MG12的 archive.php ，顾名思义这个文件是负责生成日志存档页面的，

略过获取配置等“闲言碎语”，来到了输出页面描述的地方。如下：

```php
<?php
  // If this is a category archive
  if (is_category()) {
   printf( __('Archive for the &amp;#8216;%1$s&amp;#8217; Category', 'inove'), single_cat_title('', false) );
  // If this is a tag archive
  } elseif (is_tag()) {
   printf( __('Posts Tagged &amp;#8216;%1$s&amp;#8217;', 'inove'), single_tag_title('', false) );
  // If this is a daily archive
  } elseif (is_day()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('F jS, Y', 'inove')) );
  // If this is a monthly archive
  } elseif (is_month()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('F, Y', 'inove')) );
  // If this is a yearly archive
  } elseif (is_year()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('Y', 'inove')) );
  // If this is an author archive
  } elseif (is_author()) {
   _e('Author Archive', 'inove');
  // If this is a paged archive
  } elseif (isset($_GET['paged']) &amp;&amp; !empty($_GET['paged'])) {
   _e('Blog Archives', 'inove');
  }
?>
```


MG12的写法很高级，函数套函数的，比如：

```php
  <?php printf( __('Archive for the &amp;#8216;%1$s&amp;#8217; Category', 'inove'), single_cat_title('', false) );?>
```


程序先申请两个大的变量一个存放

```php
<?php 'Archive for the &amp;#8216;%1$s&amp;#8217; Category', 'inove' ?>
```


另一个存放

```php
<?php single_cat_title('', false) ?>
```


最后再用标准的输出函数输出为

```php
<?php 'Archive for the &amp;#8216;+single_cat_title('', false)+ &amp;#8217; Category', 'inove' ?>
```

忘记从哪里看到了一个说法是printf 速度&lt;echo速度&lt;直接输出HTML速度

```php
<?php 
//于是我们可以这么写[我们是给自己用自然不用使用语言包，故删除后面的Mo文件需要的标记]

$strTmp = 'Archive for the &amp;#8216;' . single_cat_title('', false) . ' &amp;#8217; Category';

echo $strTmp;

//再来一个更快的
Archive for the “&lt;  echo single_cat_title('', false);   &gt;”Category
?>
```

这种边边角角的地方很多，如果都这样适当的提速的话[尤其是输出大段文字的时候]，相信你的iNove一定比别人的快。 

接下来我看到了一个获取标题的函数，对于这种不加参数便能在多种情况下使用的函数，我们需要特别留意，因为这种智能函数是特别浪费资源的。

```php
<?php 
  } elseif (is_day()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('F jS, Y', 'inove')) );
  // If this is a monthly archive
  } elseif (is_month()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('F, Y', 'inove')) );
  // If this is a yearly archive
  } elseif (is_year()) {
   printf( __('Archive for %1$s', 'inove'), get_the_time(__('Y', 'inove')) );
?>
```

顺藤摸瓜，在wordpress的API中找到了这个函数的原型

```php
<?php 

/**
 * Display or retrieve page title for all areas of blog.
 *
 * By default, the page title will display the separator before the page title,
 * so that the blog title will be before the page title. This is not good for
 * title display, since the blog title shows up on most tabs and not what is
 * important, which is the page that the user is looking at.
 *
 * There are also SEO benefits to having the blog title after or to the 'right'
 * or the page title. However, it is mostly common sense to have the blog title
 * to the right with most browsers supporting tabs. You can achieve this by
 * using the seplocation parameter and setting the value to 'right'. This change
 * was introduced around 2.5.0, in case backwards compatibility of themes is
 * important.
 *
 * @since 1.0.0
 *
 * @param string $sep Optional, default is '&amp;raquo;'. How to separate the various items within the page title.
 * @param bool $display Optional, default is true. Whether to display or retrieve title.
 * @param string $seplocation Optional. Direction to display title, 'right'.
 * @return string|null String on retrieve, null when displaying.
 */
function wp_title($sep = '&amp;raquo;', $display = true, $seplocation = '') {
 global $wpdb, $wp_locale, $wp_query;

 $cat = get_query_var('cat');
 $tag = get_query_var('tag_id');
 $category_name = get_query_var('category_name');
 $author = get_query_var('author');
 $author_name = get_query_var('author_name');
 $m = get_query_var('m');
 $year = get_query_var('year');
 $monthnum = get_query_var('monthnum');
 $day = get_query_var('day');
 $title = '';

 $t_sep = '%WP_TITILE_SEP%'; // Temporary separator, for accurate flipping, if necessary

 // If there's a category
 if ( !empty($cat) ) {
   // category exclusion
   if ( !stristr($cat,'-') )
    $title = apply_filters('single_cat_title', get_the_category_by_ID($cat));
 } elseif ( !empty($category_name) ) {
  if ( stristr($category_name,'/') ) {
    $category_name = explode('/',$category_name);
    if ( $category_name[count($category_name)-1] )
     $category_name = $category_name[count($category_name)-1]; // no trailing slash
    else
     $category_name = $category_name[count($category_name)-2]; // there was a trailling slash
  }
  $cat = get_term_by('slug', $category_name, 'category', OBJECT, 'display');
  if ( $cat )
   $title = apply_filters('single_cat_title', $cat-&gt;name);
 }

 if ( !empty($tag) ) {
  $tag = get_term($tag, 'post_tag', OBJECT, 'display');
  if ( is_wp_error( $tag ) )
   return $tag;
  if ( ! empty($tag-&gt;name) )
   $title = apply_filters('single_tag_title', $tag-&gt;name);
 }

 // If there's an author
 if ( !empty($author) ) {
  $title = get_userdata($author);
  $title = $title-&gt;display_name;
 }
 if ( !empty($author_name) ) {
  // We do a direct query here because we don't cache by nicename.
  $title = $wpdb-&gt;get_var($wpdb-&gt;prepare(&quot;SELECT display_name FROM $wpdb-&gt;users WHERE user_nicename = %s&quot;, $author_name));
 }

 // If there's a month
 if ( !empty($m) ) {
  $my_year = substr($m, 0, 4);
  $my_month = $wp_locale-&gt;get_month(substr($m, 4, 2));
  $my_day = intval(substr($m, 6, 2));
  $title = &quot;$my_year&quot; . ($my_month   &quot;$t_sep$my_month&quot; : &quot;&quot;) . ($my_day   &quot;$t_sep$my_day&quot; : &quot;&quot;);
 }

 if ( !empty($year) ) {
  $title = $year;
  if ( !empty($monthnum) )
   $title .= &quot;$t_sep&quot; . $wp_locale-&gt;get_month($monthnum);
  if ( !empty($day) )
   $title .= &quot;$t_sep&quot; . zeroise($day, 2);
 }

 // If there is a post
 if ( is_single() ||  ( is_page() &amp;&amp; !is_front_page() ) ) {
  $post = $wp_query-&gt;get_queried_object();
  $title = strip_tags( apply_filters( 'single_post_title', $post-&gt;post_title ) );
 }

 // If there's a taxonomy
 if ( is_tax() ) {
  $taxonomy = get_query_var( 'taxonomy' );
  $tax = get_taxonomy( $taxonomy );
  $tax = $tax-&gt;label;
  $term = $wp_query-&gt;get_queried_object();
  $term = $term-&gt;name;
  $title = &quot;$tax$t_sep$term&quot;;
 }

 if ( is_404() ) {
  $title = __('Page not found');
 }

 $prefix = '';
 if ( !empty($title) )
  $prefix = &quot; $sep &quot;;

  // Determines position of the separator and direction of the breadcrumb
 if ( 'right' == $seplocation ) { // sep on right, so reverse the order
  $title_array = explode( $t_sep, $title );
  $title_array = array_reverse( $title_array );
  $title = implode( &quot; $sep &quot;, $title_array ) . $prefix;
 } else {
  $title_array = explode( $t_sep, $title );
  $title = $prefix . implode( &quot; $sep &quot;, $title_array );
 }

 $title = apply_filters('wp_title', $title, $sep, $seplocation);

 // Send it out
 if ( $display )
  echo $title;
 else
  return $title;

}
?>
```


可以看到这个函数请求数据库多次，【继续查看其他的函数得到】，基本上前面的9个变量赋值都请求了一次或多次数据库，我们的归档页面似乎只是需要获取年月日而已吧。所以我们可以把它精简一下，以提高速度

如果你图省事的话，可以这么来，它可以像刚刚的函数一样的调用，但是只会请求三次数据库。

可是无论什么情况下都请求三次也太浪费了，我们只要获取年份，它也要获取3次...

```php
<?php 

function my_wp_archive_desc() {
 $year = get_query_var('year');
 $monthnum = get_query_var('monthnum');
 $day = get_query_var('day');
 $strRtn = '';

 if ( !empty($year) ) {
  $strRtn = $year . '年';
  if ( !empty($monthnum) ) {
   $strRtn .= $monthnum . '月';
  }
  if ( !empty($day) ) {
   $strRtn .= zeroise($day, 2) . '日';
  }
 }
  return $strRtn;
}
?>
```


于是出现了改良的版本

```php
<?php 

function my_wp_archive_descEx($mode = 3) {
 $strRtn = '';
 //请求年月日三个变量
 if ( $mode ==3 ) {
  $year = get_query_var('year');
  $monthnum = get_query_var('monthnum');
  $day = get_query_var('day');
  if ( !empty($year) ) {
   $strRtn = $year . '年';
   if ( !empty($monthnum) ) {
    $strRtn .= $monthnum . '月';
   }
   if ( !empty($day) ) {
    $strRtn .= zeroise($day, 2) . '日';
   }
  }  
 }
 //请求年月
 elseif ( $mode ==2 ) {
  $year = get_query_var('year');
  $monthnum = get_query_var('monthnum');
  if ( !empty($year) ) {
   $strRtn = $year . '年';
   if ( !empty($monthnum) ) {
    $strRtn .= $monthnum . '月';
   }
  } 
 }
 //请求年
 elseif ( $mode ==1 ){
  $year = get_query_var('year');
  if ( !empty($year) ) {
   $strRtn = $year . '年';
  }  
 }
  return $strRtn;
}
?>
```


别看代码是多了，但是并不是每个代码都要走一遍于是效率便提高了，这两个函数的使用方法很简单，如下

你可以把函数附加到你的archive.php尾部，然后替换原来的显示函数为下面的其中一种。

```php
<?php 

  } elseif (is_day()) {
   echo my_wp_archive_desc().'文章存档。';
  } elseif (is_month()) {
   echo my_wp_archive_desc().'文章存档。';
  } elseif (is_year()) {
   echo my_wp_archive_desc().'文章存档。';

//改进的

  } elseif (is_day()) {
   echo my_wp_archive_descEx(3).'文章存档。';
  } elseif (is_month()) {
   echo my_wp_archive_descEx(2).'文章存档。';
  } elseif (is_year()) {
   echo my_wp_archive_descEx(1).'文章存档。';
?>
```

待续....

