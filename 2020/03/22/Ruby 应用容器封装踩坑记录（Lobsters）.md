# Ruby 应用容器封装踩坑记录（Lobsters）

最近在基于 Lobsters 进行社区部分功能的开发，在开发过程中，需要将应用进行容器化配置和部署，经历了比较典型的 Ruby 老版本软件升级，过程中遇到了不少问题。

在此记录下，希望能帮到有相同需求的同学。

## 写在前面

首先回答为什么要考虑对 Ruby 应用进行容器化封装。

- 一来，目前线上运行的应用必须以容器方式进行交付运行，我们使用容器的方式注册应用，对外提供服务；
- 二来，个人倾向并坚持使用容器方案，可以方便后续快速水平扩展；以及最重要的一点，“代码和命令皆有记录”，方便离线的问题排查。

一般的 Web 应用封装都会经历下下几个阶段，整合源代码，安装应用依赖和环境，进行程序/产物的编译，调整权限和目录结构，进行测试，完成后对镜像打标签进行版本管理。

这次的踩坑记录亦是如此。

## 故事的开始

应用镜像的封装最早要从年前的一次模版风格定制开始，当时我们参考 [https://github.com/utensils/docker-lobsters](https://github.com/utensils/docker-lobsters) 封装了一套镜像，因为当时并未对官方程序进行依赖修改，所以用着这套镜像的程序在线上安然跑了两个多月，直至最近复工，当时的镜像文件是这样编写的：

```bash
# Lobsters
#
# VERSION latest
ARG BASE_IMAGE=ruby:2.3-alpine
FROM ${BASE_IMAGE}

# Create lobsters user and group.
RUN set -xe; \
    addgroup -S lobsters; \
    adduser -S -h /lobsters -s /bin/sh -G lobsters lobsters;

# Install needed runtime dependencies.
RUN set -xe; \
    chown -R lobsters:lobsters /lobsters; \
    apk add --no-cache --update --virtual .runtime-deps \
        mariadb-connector-c \
        bash \
        nodejs \
        npm \
        sqlite-libs \
        tzdata;

# Change shell to bash
SHELL ["/bin/bash", "-c"]

# Install needed development dependencies. If this is a developer_build we don't remove
# the build-deps after doing a bundle install.
# Copy Gemfile to container.
COPY --chown=lobsters:lobsters ./lobsters/Gemfile ./lobsters/Gemfile.lock /lobsters/
ARG DEVELOPER_BUILD=false
RUN set -xe; \
    apk add --no-cache --virtual .build-deps \
        build-base \
        curl \
        gcc \
        git \
        gnupg \
        linux-headers \
        mariadb-connector-c-dev \
        mariadb-dev \
        sqlite-dev; \
    export PATH=/lobsters/.gem/ruby/2.3.0/bin:$PATH; \
    export SUPATH=$PATH; \
    export GEM_HOME="/lobsters/.gem"; \
    export GEM_PATH="/lobsters/.gem"; \
    export BUNDLE_PATH="/lobsters/.bundle"; \
    cd /lobsters; \
    su lobsters -c "gem install bundler --user-install"; \
    su lobsters -c "gem update"; \
    su lobsters -c "gem install rake -v 12.3.2"; \
    su lobsters -c "bundle install --no-cache"; \
    su lobsters -c "bundle add puma --version '~> 3.12.1'"; \
    if [ "${DEVELOPER_BUILD,,}" != "true" ]; \
    then \
        apk del .build-deps; \
    fi; \
    mv /lobsters/Gemfile /lobsters/Gemfile.bak; \
    mv /lobsters/Gemfile.lock /lobsters/Gemfile.lock.bak;

# Copy lobsters into the container.
COPY ./lobsters ./docker-assets /lobsters/

# Set proper permissions and move assets and configs.
RUN set -xe; \
    mv /lobsters/Gemfile.bak /lobsters/Gemfile; \
    mv /lobsters/Gemfile.lock.bak /lobsters/Gemfile.lock; \
    chown -R lobsters:lobsters /lobsters; \
    mv /lobsters/docker-entrypoint.sh /usr/local/bin/; \
    chmod 755 /usr/local/bin/docker-entrypoint.sh;

# Drop down to unprivileged users
USER lobsters

# Set our working directory.
WORKDIR /lobsters/

# Build arguments.
ARG VCS_REF
ARG BUILD_DATE
ARG VERSION

# Labels / Metadata.
LABEL \
    org.opencontainers.image.authors="James Brink <brink.james@gmail.com>" \
    org.opencontainers.image.created="${BUILD_DATE}" \
    org.opencontainers.image.description="Lobsters Rails Project" \
    org.opencontainers.image.revision="${VCS_REF}" \
    org.opencontainers.image.source="https://github.com/utensils/docker-lobsters" \
    org.opencontainers.image.title="lobsters" \
    org.opencontainers.image.vendor="Utensils" \
    org.opencontainers.image.version="${VERSION}"

# Set environment variables.
ENV MARIADB_HOST="mariadb" \
    MARIADB_PORT="3306" \
    MARIADB_PASSWORD="password" \
    MARIADB_USER="root" \
    LOBSTER_DATABASE="lobsters" \
    LOBSTER_HOSTNAME="localhost" \
    LOBSTER_SITE_NAME="Example News" \
    RAILS_ENV="development" \
    SECRET_KEY="" \
    GEM_HOME="/lobsters/.gem" \
    GEM_PATH="/lobsters/.gem" \
    BUNDLE_PATH="/lobsters/.bundle" \
    RAILS_MAX_THREADS="5" \
    SMTP_HOST="127.0.0.1" \
    SMTP_PORT="25" \
    SMTP_STARTTLS_AUTO="true" \
    SMTP_USERNAME="lobsters" \
    SMTP_PASSWORD="lobsters" \
    RAILS_LOG_TO_STDOUT="1" \
    PATH="/lobsters/.gem/ruby/2.3.0/bin:$PATH"

# Expose HTTP port.
EXPOSE 3000

# Execute our entry script.
CMD ["/usr/local/bin/docker-entrypoint.sh"]
```

然而因为要对 lobsters 进行用户系统对接等修改，Gemfile / Gemfile.lock 不可避免的需要更新，开发工程师也顺手将 Ruby 版本调整到了 2.4.0 ，然而没想到只因为这么一个小小的变动，就开始了连环踩坑。

Gemfile 的变更记录其实不多：

```diff
diff --git a/Gemfile b/Gemfile
index 37f698d..ed43b5c 100644
--- a/Gemfile
+++ b/Gemfile 

@@ -11,6 +13,7 @@ gem "mysql2"
 gem 'scenic'
 gem 'scenic-mysql_adapter'
 gem "activerecord-typedstore"
+gem 'jbuilder'
 
 # js
 gem "dynamic_form"
@@ -19,9 +22,9 @@ gem "json"
 gem "uglifier", ">= 1.3.0"
 
 # deployment
-gem "actionpack-page_caching"
+gem "actionpack-page_caching", "~> 1.1.1"
 gem "exception_notification"
-gem "unicorn"
+gem "puma", "~> 4.3.3"
 
 # security
 gem "bcrypt", "~> 3.1.2"
@@ -42,6 +45,14 @@ gem 'transaction_retry' # mitigate https://github.com/lobsters/lobsters-ansible/

 gem "scout_apm", "2.6.2"
 
+gem 'settingslogic'
+
+# for oauth2
+gem 'oauth2'
+gem 'whenever', require: false
+
+gem "paranoia", "~> 2.2"
+
 group :test, :development do
   gem 'bullet'
   gem 'capybara'
@@ -57,3 +68,17 @@ group :test, :development do
   gem "byebug"
   gem "rb-readline"
 end
+
+group :development do
+  gem 'web-console', '>= 3.3.0'
+  gem 'spring'
+  gem 'spring-watcher-listen', '~> 2.0.0'
+  gem "ed25519" , "~> 1.2", require: false
+  gem "bcrypt_pbkdf", "~> 1.0", require: false
+
+  gem "capistrano",            require: false
+  gem 'capistrano-rvm',        require: false
+  gem 'capistrano-rails',      require: false
+  gem 'capistrano-bundler',    require: false
+  gem 'capistrano3-puma',      require: false
+end
```

这里需要额外提一个点，Gemfile.lock 中除了依赖更新外，bundle 版本有变化：

```diff
 BUNDLED WITH
-   2.0.2
+   1.17.3
```

基本需要关注的内容都介绍完毕了，我们先使用上面提到的 Dockerfile 进行镜像构建。

## 第一回合：尝试升级 Ruby 2.4.0

第一回合在更新镜像 Ruby 依赖时，报了版本不兼容的错误。

```TeXT
Successfully installed bundler-2.1.4
1 gem installed
+ su lobsters -c 'gem update'
ERROR:  Error installing bigdecimal:
	There are no versions of bigdecimal (= 2.0.0) compatible with your Ruby & RubyGems
	bigdecimal requires Ruby version >= 2.4.0. The current ruby version is 2.3.8.459.
ERROR:  Error installing io-console:
	There are no versions of io-console (= 0.5.6) compatible with your Ruby & RubyGems
	io-console requires Ruby version >= 2.4.0. The current ruby version is 2.3.8.459.
Updating installed gems
Updating bigdecimal
Updating bundler
Successfully installed bundler-2.1.4
Updating io-console
Updating json
Building native extensions. This could take a while...
Successfully installed json-2.3.0
Updating psych
Building native extensions. This could take a while...
ERROR:  Error installing rdoc:
	There are no versions of rdoc (= 6.2.1) compatible with your Ruby & RubyGems
	rdoc requires Ruby version >= 2.4.0. The current ruby version is 2.3.8.459.
```

考虑到实际运行环境已经升级到 ruby 2.4 ，故这里需要对容器配置文件进行修改，将 `BASE_IMAGE=ruby:2.3-alpine` 修改为 `BASE_IMAGE=ruby:2.4-alpine`，镜像配置文件中包含 `2.3.0` 的 Path 也需要更新为 `2.4.0`。

修改完毕后，我们继续下一场战斗。

### 额外的小坑：官方镜像路径

我们使用 `ruby -v` 命令可以清楚看到我们实际使用的版本是 **2.4.9p362**。

```bash
docker run --rm -it ruby:2.4-alpine ruby -v

ruby 2.4.9p362 (2019-10-02 revision 67824) [x86_64-linux-musl]
```

但是在检查本地的安装目录时，可以看到安装目录是 **2.4.0**。也就是说，官方镜像会忽略版本号最后一位修正版本号。

```bash
docker run --rm -it ruby:2.4-alpine ls /usr/local/lib/ruby/site_ruby/

2.4.0
```

所以在编写配置的时候，如果涉及定义具体路径，注意不要把修正版本写进去。

## 第二回合：手动指定 Puma 版本

将镜像升级到 `ruby:2.4-alpine` 后，经过漫长的编译等待，终于看到了熟悉的“Bundle complete! 53 Gemfile dependencies, 134 gems now installed.” 提示，说明软件依赖顺利安装完毕。

本以为这个事情就这么愉快结束了，万万没想到紧接着出现了一个经典错误，环境和实际依赖不一致：

```TeXT
Post-install message from capistrano3-puma:

    All plugins need to be explicitly installed with install_plugin.
    Please see README.md
  + su lobsters -c 'bundle add puma --version '\''~> 3.12.1'\'''

[!] There was an error parsing `injected gems`: You cannot specify the same gem twice with different version requirements.
You specified: puma (~> 4.3.3) and puma (~> 3.12.1). If you want to update the gem version, run `bundle update puma`. You may also need to change the version requirement specified in the Gemfile if it's too restrictive.. Bundler cannot continue.

 #  from injected gems:1
 #  -------------------------------------------
 >  gem "puma", "~> 3.12.1"
 #  -------------------------------------------

```

还记得之前的容器配置文件中，有一句 `su lobsters -c "bundle add puma --version '~> 3.12.1'"`命令吗？

这句命令和当前应用依赖配置中声明的 `gem "puma", "~> 4.3.3"` 冲突了。

将容器配置中的命令修改为 ` ~> 4.3.3` ，开始下一次尝试。

## 第三回合：手动指定 Rake 版本

在修改容器环境后，我们很“顺利”的将镜像打包完毕。虽然还在报类似上面的错误，但是看起来仅仅是因为软件依赖文件的声明的问题，应该不影响运行。

```TeXT
Post-install message from capistrano3-puma:

    All plugins need to be explicitly installed with install_plugin.
    Please see README.md
  + su lobsters -c 'bundle add puma --version '\''~> 4.3.3'\'''
```

倔强的尝试启动应用，会发现出现了一个新的问题 Rake 任务执行出错。

```TeXT
rake aborted!
Gem::LoadError: You have already activated rake 12.3.2, but your Gemfile requires rake 13.0.1. Prepending `bundle exec` to your command may solve this.
/lobsters/config/boot.rb:3:in `<top (required)>'
/lobsters/config/application.rb:1:in `require_relative'
/lobsters/config/application.rb:1:in `<top (required)>'
/lobsters/Rakefile:4:in `<top (required)>'
/lobsters/.gem/gems/rake-12.3.2/exe/rake:27:in `<top (required)>'
(See full trace by running task with --trace)
2020-03-21 23:26:00 - DB Version: 
2020-03-21 23:26:00 - Creating database.
rake aborted!
```

根据线索，我们在 Dockerfile 中添加一条命令，强制执行任务的 rake 软件版本。

```bash
RUN gem install rake --version 13.0.1;
```

继续新的尝试。

## 第四回合：完成 Ruby 2.4 软件运行环境

在幸运倔强下，这次软件正常运行起来了。

```TeXT
Puma starting in single mode...
* Version 4.3.3 (ruby 2.4.9-p362), codename: Mysterious Traveller
* Min threads: 5, max threads: 5
* Environment: production
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

使用 `curl` 命令验证一下程序，看到程序已经正常跑起来了。

```bash
curl https://127.0.0.1 -H "host:hub.lab.com" -I

HTTP/2 200 
```

本来到这里，应该一切就完结了，但是考虑到应用未来的可维护性，我们需要继续尝试对应用进行升级处理。

毕竟自 2.4.x 在 [2016 年末推出后](http://www.ruby-lang.org/en/news/2016/12/25/ruby-2-4-0-released/)，官方后续陆续的也出了不少[安全修复](http://www.ruby-lang.org/en/security/)，而且多数受到影响的都是老版本的 Ruby / RubyGems ，我可不想在 2020 年还在维护一个五年的软件环境，以及一堆不知道哪年推出的软件包依赖，以及他们潜在的莫名其妙的问题，和一堆已知的安全风险。

将 Dockerfile 中的 `ruby:2.4-alpine` 调整至 `ruby:2.7-alpine`，记得注意第一回合里记录的“路径细节”，再次尝试构建镜像。

## 第五回合：尝试升级 Ruby 2.7 运行环境

不出意外，又遇到了新的问题。

```TeXT
+ su lobsters -c 'bundle install --no-cache'
/usr/local/lib/ruby/2.7.0/rubygems.rb:275:in `find_spec_for_exe': Could not find 'bundler' (1.17.3) required by your /lobsters/Gemfile.lock. (Gem::GemNotFoundException)
To update to the latest version installed on your system, run `bundle update --bundler`.
To install the missing version, run `gem install bundler:1.17.3`
	from /usr/local/lib/ruby/2.7.0/rubygems.rb:294:in `activate_bin_path'
	from /lobsters/.gem/ruby/2.7.0/bin/bundle:23:in `<main>'
```

根据错误提示在镜像文件中的 `bundle install --no-cache` 前添加两条命令：

```TeXT
+ su lobsters -c "bundle update --bundler"; \
+ su lobsters -c "gem install bundler:1.17.3"; \
su lobsters -c "bundle install --no-cache"; \
```

再次构建会发现除了报告了两条警告外一切正常。

```TeXT
...
Installing whenever 1.0.0
Warning: the lockfile is being updated to Bundler 2, after which you will be unable to return to Bundler 1.
Bundle updated!
...
+ su lobsters -c 'bundle install --no-cache'
[DEPRECATED] The `--no-cache` flag is deprecated because it relies on being remembered across bundler invocations, which bundler will no longer do in future versions. Instead please use `bundle config set no-cache 'true'`, and stop using this flag
```

和第四回合一样，验证应用可以正常启动，说明修改是正确的。

但是还是存在一些问题，我们继续进行优化，解决这些不应该存在的“警告”，避免程序在运行时出现其他问题。

## 第六回合：升级 Bundler 到合适版本

迄今为止我们主要完成了下面两件事：

- 在 2.4.x 版本的 ruby 镜像中启动 lobsters
- 在 2.7.x 版本的 ruby 镜像中启动 lobsters

目前剩下的问题还有：

- 尝试升级比 ruby 2.4.x 推出时间更早的 bundler 1.7 （[2015年](https://bundler.io/v1.7/whats_new.html)），以避免后续遇到更多各种奇怪的问题
- 尝试解决各种老版本依赖、组件的潜在兼容性问题，比如 rake、puma...

上一回合中，构建镜像出现警告的根本原因在于文章开头我们指定了**BUNDLED WITH  1.17.3**。

这里推荐一个解决方案，参考 Node 和 NPM，选择跟随语言运行环境推出时间段的相关工具版本，不要 hardcode 写死版本。

其实最初的镜像文件中，其实默认就会使用 `gem` 安装最新兼容的 `bundler`。

```TeXT
...
+ su lobsters -c 'gem install bundler --user-install'
Successfully installed bundler-2.1.4
1 gem installed
+ su lobsters -c 'gem update'
Updating installed gems
Updating bundler
Successfully installed bundler-2.1.4
...
```

所以在 Gemfile.lock 中，可以直接删除 `BUNDLED WITH` 相关版本配置，另外可以将上一回合添加的安装旧版本的 `bundler` 命令从 `Dockerfile` 也删除掉。

```TeXT
su lobsters -c "bundle update --bundler"; \
su lobsters -c "gem install bundler:1.17.3"; \
```

测试构建顺利成功，启动应用也没有问题。

## 第七回合：升级 Rake 版本到合适版本

接着来解决 `rake` 的版本问题，和 `bundler` 的处理思路一样，如非必要，不需要进行额外指定是最好的。

除了第三回合我们有指定 rake 版本外，其实最初的镜像也有声明 rake 的版本。所以我们先尝试将两条声明都删除，进行镜像构建测试：

```TeXT
...
Fetching rake 13.0.1
Installing rake 13.0.1
...
```

看起来默认的 rake 版本就是 13.0.1 ，似乎是“减负成功”了。但是启动应用的时候，我们发现又有新的问题，“bundler 找不到可执行的命令”。

```TeXT
rake aborted!
Bundler::GemNotFound: Could not find rake-13.0.1 in any of the sources
...
bundler: failed to load command: rake (/usr/local/bin/rake)
Bundler::GemNotFound: Could not find rake-13.0.1 in any of the sources
...
bundler: command not found: rails
Install missing gem executables with `bundle install`
...
```

在容器镜像文件中我们有定义 `bundle install --no-cache`，所以这里错误提示后的建议的内容是不准确的，推测这里的问题是缺失 `rake` 依赖包，在镜像文件中添加命令，对其进行安装。

```bash
...
su lobsters -c "gem install bundler --user-install"; \
su lobsters -c "gem install rake";
...
```

但是报错依旧，再次看错误日志，看到一个隐藏逻辑：“rake 调用者是 bundler”，所以是不是应该先安装 rake ，再安装 bundler 呢？

将上面两条命令顺序颠倒，或者使用下面的方式合并为一条。（目前gem还是顺序安装，没有“并发安装模式”，所以下面的命令是可行的。）

```bash
su lobsters -c "gem install rake bundler --user-install";
```

果不其然，之前找不到 rake 的问题解决了，但是出现了一个新的问题。

## 第八回合：探究迷一样的 Bundler 经典报错

新出现的问题是个经典问题，程序报错形式如下：

```bash
/usr/local/lib/ruby/2.7.0/rubygems.rb:275:in `find_spec_for_exe': can't find gem rake (>= 0.a) with executable rake (Gem::GemNotFoundException)
	from /usr/local/lib/ruby/2.7.0/rubygems.rb:294:in `activate_bin_path'
	from /lobsters/.gem/ruby/2.7.0/bin/rake:23:in `<main>'
...
```

这个问题在 bundler 官方博客中有记录：[Solutions for 'Cant find gem bundler (\>= 0.a) with executable bundle'](https://bundler.io/blog/2019/05/14/solutions-for-cant-find-gem-bundler-with-executable-bundle.html)。

在官方博客文章中，有提到“The bug is fixed in RubyGems 2.7.10 or 3.0.0 and above”，理论来说我们使用的是 2.7.x 版本的最新镜像，应该是不会出现这个问题的，难道...

故技重施，查看当前使用的容器镜像中的 ruby 版本：

```bash
docker run --rm -it ruby:2.7-alpine ruby -v

ruby 2.7.0p0 (2019-12-25 revision 647ee6f091) [x86_64-linux-musl]
```

果不其然，官方镜像是“老版本”...那么我们只好尝试在容器配置文件中添加一句命令，来解决这个 bug 了。

将我们之前在容器配置文件中的命令进行升级：

```diff
- su lobsters -c 'gem update'
+ su lobsters -c 'gem update --system'
```

重新构建镜像，再次启动应用，会发现还是报相同的错误。

再次围观官方说明，会发现这个 BUG 本质是 RubyGems 和 Bundler 团队的软件约定未安装预期执行，根据官方在“Why does this bug exist?”中的说明，推测还是得在 Gemfile.lock 中指定的 Bundler 软件版本。但是实际测试，不论是在 Gemfile.lock 中声明最初的2.0.2，还是当前最新的 2.1.4 ，都无济于事。

既然版本没有达到官方文件中提到的 Ruby 2.7.10 ，根据报错行为继续推测，会不会还是环境变量中未指定路径，或者 Bundler 参数的问题呢？

在 [Bundler v2.0 官方文档](https://bundler.io/v2.0/man/bundle-install.1.html) 中找不到 `--user-install` 参数说明，但是在 [Troubleshooting common issues](https://bundler.io/doc/troubleshooting.html)中有提到这个参数仅会将软件安装至用户目录。

虽然我们在容器镜像构建时将 root 切换到 lobsters 用户，运行应用也使用的是该用户，但是说不定这个 2.7.0 版本就是根本不会读取运行用户路径下的软件呢？毕竟它身后还有至少 10 个修正版本。

```diff
su lobsters -c "gem install rake bundler --user-install"; \
su lobsters -c "gem update --system"; \

+ gem install rake; \
```

在构建过程中添加一句使用 root 用户安装 rake 至全局的命令，再次构建镜像。这里不指定版本的原因上面已经说过。

再次尝试启动镜像，一切顺利。

但是优化升级，还没有结束，我们继续战斗。

### 额外的小坑：Ruby 2.7.0 版本下 Rails 启动警告

先说结论，这个问题官方[正在解决](https://github.com/rails/rails/issues/38202)。具体情况表现为，在应用启动时会报告类似下面的警告：

```bash
/lobsters/.bundle/ruby/2.7.0/gems/activerecord-5.2.4.1/lib/active_record/migration.rb:871: warning: Using the last argument as keyword parameters is deprecated; maybe ** should be added to the call
```

如果你想让警告消失，可以采用：[How to fix Rails's warning messages with Ruby 2.7.0](https://stackoverflow.com/questions/59491848/how-to-fix-railss-warning-messages-with-ruby-2-7-0) 提到的方法。

不过个人不推荐使用非治本的方式解决问题，如果没有从本质解决问题，那么应该让问题继续暴露出来，提醒维护者后面处理掉它，而不是进行选择性遗忘。

### 额外的小坑：lockfile 和 Bundler “打架”

如果你尝试将 Bundle 指定版本降至 1.x 版本，会收到下面的错误。

```TeXT
You must use Bundler 2 or greater with this lockfile.
https://stackoverflow.com/questions/53231667/bundler-you-must-use-bundler-2-or-greater-with-this-lockfile
```

关于这个错误，[官方也有解释](https://bundler.io/blog/2019/01/04/an-update-on-the-bundler-2-release.html)，并提到："This bug was fixed in RubyGems 3.0.0 "。

果然，升级到新版本才能解决这些边边角角的奇怪问题。

## 第九回合：解决 Bundle 安装警告

第五回合结束时候，我们提到了 Bundle 的安装警告。

虽然我们在容器中首次进行安装，不需要清理缓存，但是考虑到官方镜像潜在的 tricks，还是选择设置安装时不从缓存中读取内容稳妥些。

```diff
- su lobsters -c "bundle install --no-cache"; \

+ su lobsters -c "bundle config set no-cache 'true'"; \
+ su lobsters -c "bundle install"; \
```

将配置文件参考上面的修改进行更新，再次构建镜像，这个构建过程中的安装警告果然消失了。

## 第十回合：去掉对 Puma 的版本指定

第二回合在 Ruby 2.4.0 中，我们需要指定 Puma 版本，而在 Ruby 2.7.0 中，我们可以将这句显式声明的内容删除掉，比如像下面这样修改 Dockerfile。

```diff
su lobsters -c "bundle install"; \
- su lobsters -c "bundle add puma --version '~> 4.3.3'"; \
```

为什么可以删除这条命令呢，因为在 2.7.0 的镜像容器中执行 `bundle list` 会发现当前环境已经能够根据我们的文件声明正确安装依赖了：

```bash
bundle list | grep puma

* capistrano3-puma (4.0.0)
* puma (4.3.3)
```

再次构建镜像，测试应用启动，一切正常。至此，在第六回合中我们提到的问题就都解决了。

## 第十一回合：禁止安装非必要依赖

为了可维护性，去掉不必要的冗余“代码”是很必要的。在 Gemfile 里，开发工程师定义了development 和 test 两个分组的依赖，因为容器运行在正式环境，可以避免安装这些依赖。

```bash
- su lobsters -c "bundle install"; \
+ su lobsters -c "bundle install --without=development,test"; \
```
   
对比构建结果，可以看到构建体积从 382MB 降低到了 374MB。

或许你会疑问，为什么不考虑在最初就禁用这些依赖呢？因为后续我们考虑开发环境也在容器中进行，所以需要保障带有开发依赖的配置也能够被正确初始化。

至此，让 Lobsters 正常运行在 Ruby 2.7 版本的容器中就完成了。

## 其他

如果你使用云平台的数据库产品，记得对 lobsters 使用的连接账号进行合理的授权，赋予 `ALTER` 等权限，避免应用启动时报错。

如果你也使用阿里云，则需要先登陆管理后台，再登陆数据库后台对指定用户进行授权，默认的云控制台做的太简单了，不能完成需求。

## 最后

Ruby 的构建过程是真的慢，希望有朝一日，它能够学习 Node / NPM / YARN 将一些固定环境下的编译文件进行预编译，在用户进行初始化安装的时候，能够直接提供产物，为开发者行方便，开发者也会为你提供更多有价值的回馈。

在写完这篇文章后，我对本地和服务器上进行了构建过程镜像清理，清理了大概 50 G 左右的过程产物。

--EOF