# 使用 WordPress 快速打造个人 Wiki

今年年初的时候，我曾经写过接近十篇博客，介绍如何[“从零到一搭建Wiki”](https://soulteary.com/tags/wiki.html)，聊过了 MediaWiki、Doku、Confluence、Tiddly、MoinMoin 等系统，但是这里涉及的许多系统的写作体验都不是很好。

在之后，我也写过几篇 [“如何使用容器技术搭建 WordPress”](https://soulteary.com/tags/wordpress.html) 的文章，据官方数据称 WordPress 目前已经占据了互联网 34% 的应用，写作体验和插件生态其实还是很棒的，那么我们为何不使用 WordPress 来搭建 Wiki 呢？

在使用了4个月之后，体验下来问题不大，我决定把方法分享出来，希望能帮到更多的人。如果你熟悉 WordPress 的搭建，本篇将文章两三分钟内就能够搞定啦。

## 明确目标

在搭建之前，我们明确一下个人 Wiki 的主要功能（以我个人为例）：

- 简洁好用的编辑器、能够支持 Markdown。
- 支持全文搜索、能够快速根据关键词获取参考内容。
- 支持附件管理、最好能够支持远程图片自动转存。
- 支持分类和标签功能。
- 可选的支持评论功能（单人使用可忽略）。
- 高可扩展性、但是维护成本要尽可能的低。

别看这些功能都很“基础”，文章开头提到的软件，这些功能或多或少都不支持、或者做的差强人意呢。

## 前置准备

搭建过程之前的文章中有提，有兴趣的同学可以进行翻阅。

- [使用 Docker 和 Traefik 搭建 WordPress（Nginx）](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress-with-nginx.html)
- [使用 Docker 和 Traefik 搭建 WordPress](https://soulteary.com/2019/04/07/use-docker-and-traefik-to-build-wordpress.html)

当然，你也可以使用传统的方案进行搭建。

## Wiki 界面定制

使用最新的 5.0 版本的软件，默认安装完毕后，你将看到下面的界面。

![默认的 WordPress 5.0 界面](https://attachment.soulteary.com/2019/06/28/default-ui.png)

默认的界面适合展示博客列表内容，对于 Wiki 用途而言不是特别友好，解决方案十分简单，我们进入管理后台，先将 ** 二〇一九** 主题切换为 ** Twenty Sixteen** 。

![打开管理后台主题页面](https://attachment.soulteary.com/2019/06/28/change-active-theme.png)

然后点击主题上方的“自定义”按钮，开始对主题进行自定义调整。点击左侧菜单的小工具，然后按照你的需求对主题进行调整，比如我在这里，保留了“搜索框”、“分类目录”、“近期文章”、“功能”四个模块，并按照这个顺序对模块进行了调整。

![定制侧边栏功能](https://attachment.soulteary.com/2019/06/28/select-sidebar-module.png)

接着打开 **设置**菜单中的**阅读**页面，将主页显示调整为静态页面，并选择静态页面为示例页面。

![将首页改为“静态”页面](https://attachment.soulteary.com/2019/06/28/change-home-page.png)

当前展示的页面看起来还是不像一个Wiki，那么我们继续进行调整。

还是打开管理后台的主题菜单页面，选择最后一项编辑（Theme Editor），开始对主题源文件进行修改。（如果你因为一些原因不能在浏览器直接修改这个文件，也可以通过编辑 `wp-content/themes/twentysixteen/page.php` 文件来达到同样效果）

在右侧选择 `page.php` ，原始的代码如下：

```php
<?php
/**
 * The template for displaying pages
 *
 * This is the template that displays all pages by default.
 * Please note that this is the WordPress construct of pages and that
 * other "pages" on your WordPress site will use a different template.
 *
 * @package WordPress
 * @subpackage Twenty_Sixteen
 * @since Twenty Sixteen 1.0
 */

get_header(); ?>

<div id="primary" class="content-area">
	<main id="main" class="site-main" role="main">
		<?php
		// Start the loop.
		while ( have_posts() ) :
			the_post();

			// Include the page content template.
			get_template_part( 'template-parts/content', 'page' );

			// If comments are open or we have at least one comment, load up the comment template.
			if ( comments_open() || get_comments_number() ) {
				comments_template();
			}

			// End of the loop.
		endwhile;
		?>

	</main><!-- .site-main -->

	<?php get_sidebar( 'content-bottom' ); ?>

</div><!-- .content-area -->

<?php get_sidebar(); ?>
<?php get_footer(); ?>
```

修改很简单，我们把最下面的 `get_sidebar();` 放到 `<main>` 之前，并补充一段样式即可。

```php
<?php
/**
 * The template for displaying pages
 *
 * This is the template that displays all pages by default.
 * Please note that this is the WordPress construct of pages and that
 * other "pages" on your WordPress site will use a different template.
 *
 * @package WordPress
 * @subpackage Twenty_Sixteen
 * @since Twenty Sixteen 1.0
 */

get_header(); ?>

<div id="primary" class="content-area">

	<style>
		.sidebar{
			column-count: 5;
		    column-width: 240px;
		    column-gap: 20px;
		}
	</style>

	<?php get_sidebar(); ?>

	<main id="main" class="site-main" role="main">
		<?php
		// Start the loop.
		while ( have_posts() ) :
			the_post();

			// Include the page content template.
			get_template_part( 'template-parts/content', 'page' );

			// If comments are open or we have at least one comment, load up the comment template.
			if ( comments_open() || get_comments_number() ) {
				comments_template();
			}

			// End of the loop.
		endwhile;
		?>

	</main><!-- .site-main -->

	<?php get_sidebar( 'content-bottom' ); ?>

</div><!-- .content-area -->

<?php get_footer(); ?>
```

当你在 Wiki 中适当填充一些内容之后，你会得到这样的页面。

![调整风格为类Wiki目录树](https://attachment.soulteary.com/2019/06/28/wiki-style.png)

## Markdown 语法支持

在插件中心搜索并安装 `WP Githuber MD` ，完成之后，记得启用插件。

再次打开编辑器，你会发现原本的所见即所得编辑器就变成了我们熟悉的左右分栏的 Markdown 编辑器。

![Markdown 编辑器插件界面](https://attachment.soulteary.com/2019/06/28/markdown.png)


## 代码高亮

在浏览器性能越来越高的今天，我们几乎完全不需要再使用服务端进行代码高亮啦。并且个人 Wiki 几乎没有搜索引擎 SEO 的需求。

在插件中心搜索并安装 `WP Code Highlight.js`，启用插件后，文章中的代码便会自动进行高亮展示啦。

![调整代码高亮风格](https://attachment.soulteary.com/2019/06/28/code-highlight.png)

## 用户自动登录

如果你是个人使用，搭建在内网，完全不需要考虑权限问题，那么可以和我一样，设置 WordPress 自动登录。

在 `wp-config.php` 的 `require_once ABSPATH . 'wp-settings.php';` 前，添加下面一段代码。

```php
if ($_SERVER['SCRIPT_NAME'] == '/wp-login.php') {

    require_once ABSPATH . 'wp-settings.php';
    require_once ABSPATH . '/wp-load.php';

    if (
        isset($_SERVER['QUERY_STRING']) &&
        (strpos($_SERVER['QUERY_STRING'], '=logout') !== false)
    ) {
        wp_destroy_current_session();
        wp_clear_auth_cookie();
        do_action('wp_logout');
    } else {
        $user_login = getenv('WP_USER');

        $user = get_userdatabylogin($user_login);
        $user_id = $user->ID;

        wp_set_current_user($user_id, $user_login);
        wp_set_auth_cookie($user_id);
        do_action('wp_login', $user_login);
    }
    wp_redirect(home_url(), 302);

    die;
}
```

实现原理很简单，赶在程序大部队执行前，劫持应用登陆路由，自动替用户设置登录状态。

## 最后

WordPress 是一款开源免费的软件，由 PHP 编写。前文提过，据官方数据，目前已经占据了 34% 的互联网软件份额。

> Use the software that powers over 34% of the web.
> —  https://wordpress.org/download

我个人从 2009 年开始使用它到现在：

> 在新浪云工作的时候，我负责过 WP4SAE 的开发维护，即使不看平台下载数据，单从我每次换公司，都能发现有不少的同事用过，就可以看出用户量应该还不错（偷笑）；
> 在淘宝工作的时候有写过几个下载量还不错的 WP 插件，其中一个被 360 CDN 资源站官方推荐（用于替换 Google Fonts，加速博客打开）；
> 也曾基于它（淘宝UED博客）做过一套自动化的 D2 会议电子票程序，历史 GitHub 仓库到现在还有近千 pull request 和大几百的 fork；
> 还曾在内部使用它作为 confluence 的替代者，存放多个修改版本的技术文章…

从某种意义来说，我也算是见证了这套软件的进化过程。当然，我的个人成长过程中也多次受惠于这套软件。我认为这是一款伟大的软件，某种意义来说，也是一个很成功的开源项目。

但从网上的帖子来看，一旦提起这款软件，总是出现过度的批判，甚至许多人根本不知道时过境迁，一些事情早已被改变。甚至出现了批判 WordPress 是政治正确的事情… 技术没有银弹，软件也是，在适合的场景用适合的技术，遇到问题分析并解决问题，才是技术人应该做的事情，而不是一味批判和吐槽。

感谢 WordPress ，替我节约了大量的时间去折腾更有意思的事情。

— EOF