# REMOTE_ADDR，HTTP_CLIENT_IP，HTTP_X_FORWARDED_FOR

[原文出处](http://promiseforever.com/redirect?url=http%3A%2F%2Fwww.cnblogs.com%2Flmule%2Farchive%2F2010%2F10%2F15%2F1852020.html&key=758489e8a9db2788bf57a611094e7a9e)


```php
/**
 * 获得用户的真实IP地址
 *
 * @access  public
 * @return  string
 */
function real_ip()
{
    static $realip = NULL;
 
    if ($realip !== NULL)
    {
        return $realip;
    }
 
    if (isset($_SERVER))
    {
        if (isset($_SERVER['HTTP_X_FORWARDED_FOR']))
        {
            $arr = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
 
            /* 取X-Forwarded-For中第一个非unknown的有效IP字符串 */
            foreach ($arr AS $ip)
            {
                $ip = trim($ip);
 
                if ($ip != 'unknown')
                {
                    $realip = $ip;
 
                    break;
                }
            }
        }
        elseif (isset($_SERVER['HTTP_CLIENT_IP']))
        {
            $realip = $_SERVER['HTTP_CLIENT_IP'];
        }
        else
        {
            if (isset($_SERVER['REMOTE_ADDR']))
            {
                $realip = $_SERVER['REMOTE_ADDR'];
            }
            else
            {
                $realip = '0.0.0.0';
            }
        }
    }
    else
    {
        if (getenv('HTTP_X_FORWARDED_FOR'))
        {
            $realip = getenv('HTTP_X_FORWARDED_FOR');
        }
        elseif (getenv('HTTP_CLIENT_IP'))
        {
            $realip = getenv('HTTP_CLIENT_IP');
        }
        else
        {
            $realip = getenv('REMOTE_ADDR');
        }
    }
 
    preg_match("/[\d\.]{7,15}/", $realip, $onlineip);
    $realip = !empty($onlineip[0]) ? $onlineip[0] : '0.0.0.0';
 
    return $realip;
}
```

<!-- more -->

顺便说下$_SERVER和getenv的区别，getenv不支持IIS的isapi方式运行的php

一、没有使用代理服务器的情况：
 
      REMOTE_ADDR = 您的 IP
      HTTP_VIA = 没数值或不显示
      HTTP_X_FORWARDED_FOR = 没数值或不显示

二、使用透明代理服务器的情况：Transparent Proxies

      REMOTE_ADDR = 最后一个代理服务器 IP
      HTTP_VIA = 代理服务器 IP
      HTTP_X_FORWARDED_FOR = 您的真实 IP ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。

  这类代理服务器还是将您的信息转发给您的访问对象，无法达到隐藏真实身份的目的。

三、使用普通匿名代理服务器的情况：Anonymous Proxies

      REMOTE_ADDR = 最后一个代理服务器 IP
      HTTP_VIA = 代理服务器 IP
      HTTP_X_FORWARDED_FOR = 代理服务器 IP ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。

  隐藏了您的真实IP，但是向访问对象透露了您是使用代理服务器访问他们的。

四、使用欺骗性代理服务器的情况：Distorting Proxies

      REMOTE_ADDR = 代理服务器 IP
      HTTP_VIA = 代理服务器 IP
      HTTP_X_FORWARDED_FOR = 随机的 IP ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。

  告诉了访问对象您使用了代理服务器，但编造了一个虚假的随机IP代替您的真实IP欺骗它。

五、使用高匿名代理服务器的情况：High Anonymity Proxies (Elite proxies)
 
      REMOTE_ADDR = 代理服务器 IP
      HTTP_VIA = 没数值或不显示
      HTTP_X_FORWARDED_FOR = 没数值或不显示 ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。

  完全用代理服务器的信息替代了您的所有信息，就象您就是完全使用那台代理服务器直接访问对象。

REMOTE_ADDR 是你的客户端跟你的服务器“握手”时候的IP。如果使用了“匿名代理”，REMOTE_ADDR将显示代理服务器的IP。
HTTP_CLIENT_IP 是代理服务器发送的HTTP头。如果是“超级匿名代理”，则返回none值。

同样，REMOTE_ADDR也会被替换为这个代理服务器的IP。
$_SERVER['REMOTE_ADDR']; //访问端（有可能是用户，有可能是代理的）IP
$_SERVER['HTTP_CLIENT_IP'];  //代理端的（有可能存在，可伪造）
$_SERVER['HTTP_X_FORWARDED_FOR']; //用户是在哪个IP使用的代理（有可能存在，也可以伪造）
 
－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
在WEB开发中.我们可能都习惯使用下面的代码来获取客户端的IP地址: 
C#代码
复制代码 代码如下:

//优先取得代理IP 
string IP = Request.ServerVariables["HTTP_X_FORWARDED_FOR"]; 
if (string.IsNullOrEmpty(IP)) { 
//没有代理IP则直接取连接客户端IP 
IP = Request.ServerVariables["REMOTE_ADDR"]; 
}

上面代码看来起是正常的.可惜这里却隐藏了一个隐患!!因为"HTTP_X_FORWARDED_FOR"这个值是通过获取HTTP头的"X_FORWARDED_FOR"属性取得.所以这里就提供给恶意破坏者一个办法:可以伪造IP地址!! 
下面是测试代码:
复制代码 代码如下:

HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create("http://localhost/ip.aspx"); 
request.Headers.Add("X_FORWARDED_FOR", "0.0.0.0"); 
HttpWebResponse response = (HttpWebResponse)request.GetResponse(); 
StreamReader stream = new StreamReader(response.GetResponseStream()); 
string IP = stream.ReadToEnd(); 
stream.Close(); 
response.Close(); 
request = null;

"ip.aspx"文件代码:
复制代码 代码如下:

Response.Clear(); 
//优先取得代理IP 
string IP = Request.ServerVariables["HTTP_X_FORWARDED_FOR"]; 
if (string.IsNullOrEmpty(IP)) 
{ 
//没有代理IP则直接取客户端IP 
IP = Request.ServerVariables["REMOTE_ADDR"]; 
} 
Response.Write(IP); 
Response.End();

这样.当测试代码中去访问ip.aspx文件时."string IP = stream.ReadToEnd();"这段代码取到的IP数据就是"0.0.0.0"!!!!(呵.在真实情况下.这样的IP地址肯定不是我们想要的结果.而在有些投票系统中限制一个IP只能投1次票时,如果也是用类似的代码取得对方IP然后再判断的话.呵呵.限制就失效咯)...
 
或者如果你用上面代码获取IP地址后后面又不再进行数据判断的话也许还能更进一步进行数据破坏!! 
比如你用类似上面的代码中获取IP地址就直接有这样的SQL语句: 
string sql = "INSERT INTO (IP) VALUE ('" + IP + "')"; 
那么也许破坏者还可以进行SQL注入进行数据破坏!!
这样看来利用"HTTP_X_FORWARDED_FOR"这个属性获取客户端IP的方法就不再可取了.-_-# 但如果不用这种方法.那么那些真正使用了代理服务器的人.我们又不能再获取到他们的真实IP地址(因为某些代理服务器会在"X_FORWARDED_FOR"这个HTTP头里加上访问用户真正的IP地址).呵.现实就是这样,某种东西都有有得必有失...
－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
看了这两个帖子，终于明白为什么获取ip之后还要对其正则验证了，象我以前那种直接通过REMOTE_ADDR获取客户端ip，并且不对齐进行验证的想法是很傻很天真的，必须严厉打击！

