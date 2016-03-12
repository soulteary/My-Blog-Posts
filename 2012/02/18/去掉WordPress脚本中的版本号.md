# 去掉WordPress脚本中的版本号

WordPress为了能给用户提供最新的脚本以及样式表的版本，使用回调函数生成相关HTML。但是这样生成的HTML不能实现缓存,而且不利于SEO结构化,那么我们来去掉这些讨厌的静态资源中的版本号吧.

<!-- more -->

你常常能看到这个吧

```
<script type="text/javascript" src="http://你的网站/wp-includes/js/jquery/jquery.js?=1.71"></script>
```

如果你也觉得这个版本号碍事，而又不想影响wordpress其他功能的话，只需要小小修改一下，就能解决问题了。
打开wp-includes/class.wp-scripts.php文件,找到此处

```php
if ( isset($this->args[$handle]) )
	$ver = $ver ? $ver . '&amp;' . $this->args[$handle] : $this->args[$handle];
	$src = $this->registered[$handle]->src;
```

你可以在那行代码下添加一个新的赋值语句，也可以把上面2行判断直接删除。

```php
if ( isset($this->args[$handle]) )
	$ver = $ver ? $ver . '&amp;' . $this->args[$handle] : $this->args[$handle];
$ver ='';
	$src = $this->registered[$handle]->src;
```

至此，wordpress生成的脚本，样式代码中就不会出现版本号了。

如果你愿意的话,可以用我修改过的函数替换掉你原来的函数

```php
	function do_item( $handle, $group = false ) {
		if ( !parent::do_item($handle) )
			return false;

		if ( 0 === $group && $this->groups[$handle] > 0 ) {
			$this->in_footer[] = $handle;
			return false;
		}

		if ( false === $group && in_array($handle, $this->in_footer, true) )
			$this->in_footer = array_diff( $this->in_footer, (array) $handle );

		$src = $this->registered[$handle]->src;

		if ( $this->do_concat ) {
			$srce = apply_filters( 'script_loader_src', $src, $handle );
			if ( $this->in_default_dir($srce) ) {
				$this->print_code .= $this->print_extra_script( $handle, false );
				$this->concat .= "$handle,";
				$this->concat_version .= "$handle";
				return true;
			} else {
				$this->ext_handles .= "$handle,";
				$this->ext_version .= "$handle";
			}
		}

		$this->print_extra_script( $handle );
		if ( !preg_match('|^https?://|', $src) && ! ( $this->content_url && 0 === strpos($src, $this->content_url) ) ) {
			$src = $this->base_url . $src;
		}


		$src = esc_url( apply_filters( 'script_loader_src', $src, $handle ) );

		if ( $this->do_concat )
			$this->print_html .= "<script type='text/javascript' src='$src'></script>\n";
		else
			echo "<script type='text/javascript' src='$src'></script>\n";

		return true;
	}
```

样式表后面的版本号比如style.css?3.3也可以这么去掉,修改class.wp-styles.php即可


```php
<?php
/**
 * BackPress Styles enqueue.
 *
 * These classes were refactored from the WordPress WP_Scripts and WordPress
 * script enqueue API.
 *
 * @package BackPress
 * @since r74
 */

/**
 * BackPress Styles enqueue class.
 *
 * @package BackPress
 * @uses WP_Dependencies
 * @since r74
 */
class WP_Styles extends WP_Dependencies {
	var $base_url;
	var $content_url;
	var $default_version;
	var $text_direction = 'ltr';
	var $concat = '';
	var $concat_version = '';
	var $do_concat = false;
	var $print_html = '';
	var $print_code = '';
	var $default_dirs;

	function __construct() {
		do_action_ref_array( 'wp_default_styles', array(&$this) );
	}

	function do_item( $handle ) {
		if ( !parent::do_item($handle) )
			return false;

		$obj = $this->registered[$handle];

		if ( $this->do_concat ) {
			if ( $this->in_default_dir($obj->src) && !isset($obj->extra['conditional']) && !isset($obj->extra['alt']) ) {
				$this->concat .= "$handle,";
				$this->concat_version .= "$handle";

				$this->print_code .= $this->get_data( $handle, 'after' );

				return true;
			}
		}

		if ( isset($obj->args) )
			$media = esc_attr( $obj->args );
		else
			$media = 'all';

		$href = $this->_css_href( $obj->src, $handle );
		$rel = isset($obj->extra['alt']) && $obj->extra['alt'] ? 'alternate stylesheet' : 'stylesheet';
		$title = isset($obj->extra['title']) ? "title='" . esc_attr( $obj->extra['title'] ) . "'" : '';

		$end_cond = $tag = '';
		if ( isset($obj->extra['conditional']) && $obj->extra['conditional'] ) {
			$tag .= "<!--[if {$obj->extra['conditional']}]>\n";
			$end_cond = "<![endif]-->\n";
		}

		$tag .= apply_filters( 'style_loader_tag', "<link rel='$rel' id='$handle-css' $title href='$href' type='text/css' media='$media' />\n", $handle );
		if ( 'rtl' === $this->text_direction && isset($obj->extra['rtl']) && $obj->extra['rtl'] ) {
			if ( is_bool( $obj->extra['rtl'] ) ) {
				$suffix = isset( $obj->extra['suffix'] ) ? $obj->extra['suffix'] : '';
				$rtl_href = str_replace( "{$suffix}.css", "-rtl{$suffix}.css", $this->_css_href( $obj->src , "$handle-rtl" ));
			} else {
				$rtl_href = $this->_css_href( $obj->extra['rtl'], "$handle-rtl" );
			}

			$tag .= apply_filters( 'style_loader_tag', "<link rel='$rel' id='$handle-rtl-css' $title href='$rtl_href' type='text/css' media='$media' />\n", $handle );
		}

		$tag .= $end_cond;

		if ( $this->do_concat ) {
			$this->print_html .= $tag;
			$this->print_html .= $this->print_inline_style( $handle, false );
		} else {
			echo $tag;
			$this->print_inline_style( $handle );
		}

		return true;
	}

	function add_inline_style( $handle, $code ) {
		if ( !$code )
			return false;

		$after = $this->get_data( $handle, 'after' );
		if ( !$after )
			$after = array();

		$after[] = $code;

		return $this->add_data( $handle, 'after', $after );
	}

	function print_inline_style( $handle, $echo = true ) {
		$output = $this->get_data( $handle, 'after' );

		if ( empty( $output ) )
			return false;

		$output = implode( "\n", $output );

		if ( !$echo )
			return $output;

		echo "<style type='text/css'>\n";
		echo "$output\n";
		echo "</style>\n";

		return true;
	}

	function all_deps( $handles, $recursion = false, $group = false ) {
		$r = parent::all_deps( $handles, $recursion );
		if ( !$recursion )
			$this->to_do = apply_filters( 'print_styles_array', $this->to_do );
		return $r;
	}

	function _css_href( $src, $handle ) {
		if ( !is_bool($src) && !preg_match('|^https?://|', $src) && ! ( $this->content_url && 0 === strpos($src, $this->content_url) ) ) {
			$src = $this->base_url . $src;
		}

		$src = apply_filters( 'style_loader_src', $src, $handle );
		return esc_url( $src );
	}

	function in_default_dir($src) {
		if ( ! $this->default_dirs )
			return true;

		foreach ( (array) $this->default_dirs as $test ) {
			if ( 0 === strpos($src, $test) )
				return true;
		}
		return false;
	}

	function do_footer_items() { // HTML 5 allows styles in the body, grab late enqueued items and output them in the footer.
		$this->do_items(false, 1);
		return $this->done;
	}

	function reset() {
		$this->do_concat = false;
		$this->concat = '';
		$this->concat_version = '';
		$this->print_html = '';
	}
}
```

修改后的wp css类


