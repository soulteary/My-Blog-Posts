# 简单配置服务端代理Tengine

刚刚说完[Apache](http://www.soulteary.com/2015/01/24/configure-apache.html)，接下来写一下tengine(nginx)。tengine是建立在nginx上的开源软件，添加了一大堆feature，并且你可以使用自定义的内存管理，不管是作为前端代理，还是前端缓存，效果都是萌萌哒的。 nginx和tengine略有差异，请查看[官方Wiki](http://wiki.nginx.org/Main)、[Tengine](http://tengine.taobao.org/documentation_cn.html)。

```nginx
## 根据自己情况选择用户
user  nobody;
## 建议设置机器CPU核心数目
worker_processes  1;
## 之前配置机器的时候设置过的打开数目
worker_rlimit_nofile 51200;

## 记录错误日志
error_log  logs/error.log;
## pid文件
pid        logs/nginx.pid;

## 设置最大连接数和使用epoll提高效率
events {
    worker_connections  1024;
    use epoll;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    ## 打底配置日志
    access_log  logs/access.log;
    ## 基础限流
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    limit_req_log_level error;
    ## 允许使用sendfile
    sendfile        on;
    ## 配合sendfile
    tcp_nopush     on;
    ## 关闭基础信息
    server_info off;
    ## 根据自己的站点设置
    keepalive_timeout  50;
    ## 开启gzip，这里如果不考虑IE6也没关系的话，不需要根据UA关闭GZIP
    gzip  on;
    gzip_proxied any;
    gzip_clear_etag on;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/javascript;
    ## 加载站点配置
    include vhosts/*;
}
```

和apache一样，简单阻止一些访问，根据自己的情况添加和修改。

```nginx
# robots.txt 不进行记录
location = /robots.txt { access_log off; log_not_found off; }
# favicon.ico 不进行记录
location = /favicon.ico { access_log off; log_not_found off; }
# 隐藏文件 不进行记录且禁止访问
location ~ /\. { access_log off; log_not_found off; deny all; }
# 不存在的备份文件 不进行记录且禁止访问
location ~* "bbs\.zip" { access_log off; log_not_found off; deny all; }
location ~* "wwwroot\.zip" { access_log off; log_not_found off; deny all; }
location ~* ".*\.asp$|.*\.aspx$|.*\.jsp$|.*\.mdb|.*\.log" { access_log off; log_not_found off; deny all; }
location ~* "fckeditor|ckfinder|~root" { access_log off; log_not_found off; deny all; }

# ~结尾文件不进行记录且禁止访问
location ~ ~$ { access_log off; log_not_found off; deny all; }
# 设置常用文件缓存为30天
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$ { expires 30d; }

##
# 阻止注入的一些设置
##
location ~* "union.*select.*\(|union.*all.*select.*|concat.*\(" { deny  all; }

##
# 阻止常规利用的一些设置
##
location ~* "proc/self/environ" { deny  all; }

##
# 阻止垃圾评论的一些设置
##
if ($http_user_agent ~ "\b(ultram|unicauca|valium|viagra|vicodin|xanax|ypxaieo|erections|hoodia|huronriveracres|impotence|levitra|libido|ambien|blue\spill|cialis|cocaine|ejaculation|erectile|lipitor|phentermin|pro[sz]ac|sandyauer|tramadol|troyhamby)\b") { return 404; }

##
# 阻止UA的一些设置
##
set $block_user_agents 0;
# 拒绝无UA访问
if ($http_user_agent ~ "^$") { set $block_user_agents 1; }
# 根据自己情况拒绝wget以及curl
if ($http_user_agent ~ "Wget|wget|curl|libwww-perl|httplib|WordPress|wordpress|PycURL|POE::Component::Client|InetURL|Microsoft URL Control") { set $block_user_agents 1; }
if ($http_user_agent ~ "WebCopier|Offline Explorer|Sphider|mail") { set $block_user_agents 1; }
if ($http_user_agent ~ "Opera/9\.0 \(Windows NT 5\.1; U; en\)|Opera/6\.01 \(Windows ME; U\) \[en\]") { set $block_user_agents 1; }
if ($http_user_agent ~ "Mozilla/3\.0") { set $block_user_agents 1; }
if ($http_user_agent ~ "DotBot|YandexBot|Superfeedr") { set $block_user_agents 1; }
if ($block_user_agents = 1) { return 404; }
```

对于默认IP的访问的处理

```nginx
##
# 默认IP地址
##
server {
    listen       80 default;
    server_name  _;

    location / {
        # 允许哪些
        if ( 自己的条件 ){
            return 200;
        }
        # 禁止直接访问IP地址
        return 444;
    }
}
```

简单配置访问规则，配合hhvm之类的fast-cgi的程序使用，省略配置目录反代等。 这里apache和nginx有个细节差异，apache rewrite -L是强制转向，nginx如果要实现隐性301必须使用代理模式。

```nginx
## soulteary.com www.soulteary.com
server {
    listen 80;
    server_name soulteary.com www.soulteary.com;
    ## 如果做了数据分离，可以去掉。
    ## client_max_body_size 10m;

    access_log /yourpath/www.soulteary.com/logs/access.log;
    error_log /yourpath/www.soulteary.com/logs/error.log;

    server_name_in_redirect off;
    include nginx-security.conf;

    root /yourpath/www.soulteary.com/public;
    index index.php index.html index.htm;

    location ~ /\.(gif|jpg|png|css|js|ico|swf|svg)$ {
        expires max;
    }

    location / {
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }

    location ~ \.(hh|php)$ {
        fastcgi_keep_conn on;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

使用nginx作为node前端转发时的设定。 当然，如果你的node直接跑在最前端，那么请适当修改，对于IP地址直接取remoteAddress，不要相信转发。

```nginx
upstream ghost_soulteary_upstream {
        server 127.0.0.1:2378;
        keepalive 64;
}

server {
    listen 80;
    server_name www.soulteary.im soulteary.im;
    if_modified_since before;

    server_name_in_redirect off;
    include nginx-node-security.conf;

    location / {
        proxy_cache_valid 200 30m;
        proxy_cache_valid 404 1m;
        proxy_pass http://ghost_soulteary_upstream;
        proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
        proxy_ignore_headers Set-Cookie;
        proxy_hide_header Set-Cookie;
        proxy_hide_header X-powered-by;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        expires 10m;
    }
}
```

如果你只是需要一个简单的静态站点的话，可以使用如下配置：

```nginx
## www.soulteary.com
server {
    listen 80;
    server_name www.soulteary.com;

    access_log /yourpath/www.soulteary.com/logs/access.log;
    error_log /yourpath/www.soulteary.com/logs/error.log;

    server_name_in_redirect off;
    include nginx-security.conf;

    valid_referers none blocked server_names *.soulteary.com soulteary.com;
    if ($invalid_referer) {
        rewrite ^/ "http://www.baidu.com/s?wd=妈妈说不要盗链" last;
        return 404;
    }

    root /yourpath/www.soulteary.com/public;
    index index.html;
}
```

接下来写写network/redis/hhvm/ghost的设置吧。

