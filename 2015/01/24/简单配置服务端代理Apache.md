# 简单配置服务端代理Apache

nginx和apache各有千秋，前者专注前端代理，后者生态圈有大量协同软件，两者交叉的圈子的前端代理行为也有诸多细节不同，个人建议，如果是开源软件，请本地使用apache作为主代理环境，远程服务端和本地测试端使用nginx作为测试环境，以做到兼容不同的前端代理（重写，探测等功能点）。

简单说明一下两者勾搭HHVM/Node，以及Anit，先写apache吧。

有部分线上环境，甚至会使用nginx作为apache前端，不过如果本地测试（非模拟），不必如此。 [apache文档](http://httpd.apache.org/docs/trunk/)，如果你用的不是trunk版，请选择自己的版本。 

先发本地Apache配置示例： 

主配置文件因人而异，如果需要重写，勿忘开启。

```text
LoadModule rewrite_module modules/mod_rewrite.so
```

不建议内容都写在主目录中，请使用子文件引入：

```text
# Virtual hosts
Include etc/extra/httpd-vhosts.conf
# Various default settings
Include etc/extra/httpd-default.conf
```

默认打底配置，超时时间如果是下载机，请适当调整。

```text
#
# This configuration file reflects default settings for Apache HTTP Server.
#
# You may change these, but chances are that you may not need to.
#

#
# Timeout: The number of seconds before receives and sends time out.
#
Timeout 55

#
# KeepAlive: Whether or not to allow persistent connections (more than
# one request per connection). Set to "Off" to deactivate.
#
KeepAlive On

#
# MaxKeepAliveRequests: The maximum number of requests to allow
# during a persistent connection. Set to 0 to allow an unlimited amount.
# We recommend you leave this number high, for maximum performance.
#
MaxKeepAliveRequests 100

#
# KeepAliveTimeout: Number of seconds to wait for the next request from the
# same client on the same connection.
#
KeepAliveTimeout 10

#
# UseCanonicalName: Determines how Apache constructs self-referencing 
# URLs and the SERVER_NAME and SERVER_PORT variables.
# When set "Off", Apache will use the Hostname and Port supplied
# by the client.  When set "On", Apache will use the value of the
# ServerName directive.
#
UseCanonicalName Off

#
# AccessFileName: The name of the file to look for in each directory
# for additional configuration directives.  See also the AllowOverride 
# directive.
#
AccessFileName .htaccess

#
# ServerTokens
# This directive configures what you return as the Server HTTP response
# Header. The default is 'Full' which sends information about the OS-Type
# and compiled in modules.
# Set to one of:  Full | OS | Minor | Minimal | Major | Prod
# where Full conveys the most information, and Prod the least.
#
ServerTokens Prod

#
# Optionally add a line containing the server version and virtual host
# name to server-generated pages (internal error documents, FTP directory 
# listings, mod_status and mod_info output etc., but not CGI generated 
# documents or custom error documents).
# Set to "EMail" to also include a mailto: link to the ServerAdmin.
# Set to one of:  On | Off | EMail
#
ServerSignature Off

#
# HostnameLookups: Log the names of clients or just their IP addresses
# e.g., www.apache.org (on) or 204.62.129.132 (off).
# The default is off because it'd be overall better for the net if people
# had to knowingly turn this feature on, since enabling it means that
# each client request will result in AT LEAST one lookup request to the
# nameserver.
#
HostnameLookups Off

#
# Set a timeout for how long the client may take to send the request header
# and body.
# The default for the headers is header=20-40,MinRate=500, which means wait
# for the first byte of headers for 20 seconds. If some data arrives,
# increase the timeout corresponding to a data rate of 500 bytes/s, but not
# above 40 seconds.
# The default for the request body is body=20,MinRate=500, which is the same
# but has no upper limit for the timeout.
# To disable, set to header=0 body=0
#
<IfModule reqtimeout_module>
  RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
</IfModule>
```

如果你的站点没有使用fast—cgi方式的话，可以这样设置：

```text
## www.soulteary.com
<VirtualHost *:80>
    <Directory "/Users/soulteary/Blog/www.soulteary.com/public">
        Options Indexes FollowSymLinks ExecCGI Includes
        AllowOverride All
        Require all granted
    </Directory>

    <IfModule dir_module>
        DirectoryIndex index.php index.html
    </IfModule>

    <Files ".ht*">
        Require all denied
    </Files>

    # 如果你需要调整 mime-type，可以这样
    # AddType text/x-component .htc

    LogLevel warn

    ServerAdmin webmaster@soulteary.com
    DocumentRoot "/Users/soulteary/Blog/www.soulteary.com/public"
    ServerName www.soulteary.com
    ServerAlias soulteary.com
    ErrorLog "/Users/soulteary/Blog/www.soulteary.com/logs/www.soulteary.com-error.log"
    CustomLog "/Users/soulteary/Blog/www.soulteary.com/logs/www.soulteary.com-access.log" combined
</VirtualHost>
```

如果你要配合hhvm之类的fast-cgi的程序使用

```text
## www.soulteary.com
<VirtualHost *:80>
    ServerAdmin developer@www.soulteary.com
    DocumentRoot "/yourpath/www.soulteary.com/public"
    ServerName www.soulteary.com
    ServerAlias soulteary.com

    <Directory "/yourpath/www.soulteary.com/public">
        Options Indexes FollowSymLinks ExecCGI Includes
        Options -MultiViews
        AllowOverride All
        Require all granted
        DirectoryIndex index.php index.html
    </Directory>

    ErrorLog "/yourpath/www.soulteary.com/logs/www.soulteary.com-error_log"
    CustomLog "/yourpath/www.soulteary.com/logs/www.soulteary.com-access_log" common

    <IfModule mod_proxy.c>
        ProxyPass / fcgi://127.0.0.1:9000/yourpath/www.soulteary.com/public/
        ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/yourpath/www.soulteary.com/public/$1
        ProxyVia Off
    </IfModule>

</VirtualHost>
```

默认站点以IP访问的时候你可以禁止访问，也可以允许指定的IP或者UA访问（比如linode logview等），如果是资讯类站点，可以考虑把流量引导站点内。

```apache
## 默认站点禁止访问
<VirtualHost *:80>
    ## 你的服务器监听的IP地址
    ServerName 12.34.56.78
    ## 服务器有多个IP？
    #ServerAliase 12.34.56.79
    ## 写一个默认的，以免牵扯出其他站点的信息
    ServerAdmin webmaster@localhost
    ## 限定一个目录
    DocumentRoot /var/www/html
    <Directory />
        Order deny,allow
        Deny from all
    </Directory>

    ## 你可以转向访问到你的站点，并附带一个尾巴
    RedirectMatch ^/.*$ http://www.soulteary.com/?ref=ip
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

关于Apache 有一个配置细节，就是因为程序是顺序解析执行，需要先定义document directory，在定义index 文件，防止程序自己找不到报错。 有的童鞋经常看到自己的站点被垃圾蜘蛛或者扫描器爬，不妨添加一些简单的规则，去过滤这类爬取。

```text
# blocking bots, spammers, harvesters...
SetEnvIfNoCase User-Agent "Download Ninja 2.0" block_bad_bots
SetEnvIfNoCase User-Agent "Fetch API Request" block_bad_bots
SetEnvIfNoCase User-Agent "HTTrack" block_bad_bots
SetEnvIfNoCase User-Agent "ia_archiver" block_bad_bots
SetEnvIfNoCase User-Agent "JBH Agent 2.0" block_bad_bots
SetEnvIfNoCase User-Agent "QuepasaCreep" block_bad_bots
SetEnvIfNoCase User-Agent "Program Shareware 1.0.0" block_bad_bots
SetEnvIfNoCase User-Agent "TestBED.6.3" block_bad_bots
SetEnvIfNoCase User-Agent "WebAuto" block_bad_bots
SetEnvIfNoCase User-Agent "WebCopier" block_bad_bots
SetEnvIfNoCase User-Agent "Wget/1.8.2" block_bad_bots
SetEnvIfNoCase User-Agent "Offline Explorer" block_bad_bots
SetEnvIfNoCase User-Agent "Franklin Locator" block_bad_bots
SetEnvIfNoCase User-Agent "LWP::Simple" block_bad_bots
SetEnvIfNoCase User-Agent "Larbin" block_bad_bots
SetEnvIfNoCase User-Agent "Rufus Web Miner" block_bad_bots
SetEnvIfNoCase User-Agent "Port Huron Labs" block_bad_bots
SetEnvIfNoCase User-Agent "Sphider" block_bad_bots
SetEnvIfNoCase User-Agent "voyager/1.0" block_bad_bots
SetEnvIfNoCase User-Agent "DynaWeb" block_bad_bots
SetEnvIfNoCase User-Agent "EmailCollector/1.0" block_bad_bots_bot
SetEnvIfNoCase User-Agent "EmailSiphon" block_bad_bots_bot
SetEnvIfNoCase User-Agent "EmailWolf 1.00" block_bad_bots_bot
SetEnvIfNoCase User-Agent "ExtractorPro" block_bad_bots_bot
SetEnvIfNoCase User-Agent "Crescent Internet ToolPak" block_bad_bots_bot
SetEnvIfNoCase User-Agent "CherryPicker/1.0" block_bad_bots_bot
SetEnvIfNoCase User-Agent "CherryPickerSE/1.0" block_bad_bots_bot
SetEnvIfNoCase User-Agent "CherryPickerElite/1.0" block_bad_bots_bot
SetEnvIfNoCase User-Agent "NICErsPRO" block_bad_bots_bot
SetEnvIfNoCase User-Agent "WebBandit/2.1" block_bad_bots_bot
SetEnvIfNoCase User-Agent "WebBandit/3.50" block_bad_bots_bot
SetEnvIfNoCase User-Agent "webbandit/4.00.0" block_bad_bots_bot
SetEnvIfNoCase User-Agent "WebEMailExtractor/1.0B" block_bad_bots_bot
SetEnvIfNoCase User-Agent "autoemailspider" block_bad_bots_bot
SetEnvIfNoCase User-Agent "^libwww-perl" block_bad_bots
SetEnvIfNoCase User-Agent "^WordPress/2\.0\.2" block_bad_bots
SetEnvIfNoCase User-Agent "^Opera/9\.0 \(Windows NT 5\.1; U; en\)" block_bad_bots
SetEnvIfNoCase User-Agent "^PycURL/7\.15\.5$" block_bad_bots
SetEnvIfNoCase User-Agent "^TurnitinBot" block_bad_bots
SetEnvIfNoCase User-Agent "^West Wind Internet Protocols" block_bad_bots
SetEnvIfNoCase User-Agent "^POE::Component::Client::HTTP/" block_bad_bots
SetEnvIfNoCase User-Agent "^User-Agent: Mozilla/4.0" block_bad_bots
SetEnvIfNoCase User-Agent "netforex" block_bad_bots
SetEnvIfNoCase User-Agent "^SMBot/" block_bad_bots
SetEnvIfNoCase User-Agent "^Mozilla/4.0 \(compatible; MSIE 4\.0; Windows NT; \.\.\.\.\.\./1\.0 \)$" block_bad_bots
SetEnvIfNoCase User-Agent "envolk" block_bad_bots
SetEnvIfNoCase User-Agent "^TMCrawler" block_bad_bots
SetEnvIfNoCase User-Agent "^Opera/6\.01 \(Windows ME; U\) \[en\]" block_bad_bots
SetEnvIfNoCase User-Agent "^NASA Search" block_bad_bots
SetEnvIfNoCase User-Agent "^TrackBack/" block_bad_bots
SetEnvIfNoCase User-Agent "^Mozilla/4\.0 \(compatible; MSIE 6\.0; Windows NT 5\.0; Maxthon\)$" block_bad_bots
SetEnvIfNoCase User-Agent "QihooBot" block_bad_bots
SetEnvIfNoCase User-Agent "^Gigabot/.\.." block_bad_bots
SetEnvIfNoCase User-Agent "K-Meleon/0\.8" block_bad_bots
SetEnvIfNoCase User-Agent "Twiceler" block_bad_bots
SetEnvIfNoCase User-Agent "^NSPlayer" block_bad_bots
SetEnvIfNoCase User-Agent "^Microsoft URL Control" block_bad_bots
SetEnvIfNoCase User-Agent "^InetURL" block_bad_bots
SetEnvIfNoCase User-Agent "DotBot" block_bad_bots
SetEnvIfNoCase User-Agent "YandexBot" block_bad_bots
SetEnvIfNoCase User-Agent "Nutch" block_bad_bots
SetEnvIfNoCase User-Agent "Mozilla/4\.0 \(compatible; ICS\)" block_bad_bots
SetEnvIfNoCase User-Agent "Mozilla/3\.0 \(compatible; Indy Library\)" block_bad_bots

<Directory /var/www/vhosts/>
Order Allow,Deny
Allow from all
Deny from env=block_bad_bots
</Directory>
```
