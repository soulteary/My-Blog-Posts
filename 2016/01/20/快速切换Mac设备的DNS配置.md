# 快速切换Mac设备的DNS配置

在WEB开发过程经常遇到调试需求，而调试又偏偏依赖网络环境的时候，这个时候来回绑定HOST或者切换DNS未免枯燥。

前厂的童鞋有写iHost来一键切换配置，当然，类似方案挺多的，但是便捷的切换DNS服务器和搜索域的工具就不多了。

## 当前问题

*   浏览器有插件可以自动切换环境，但是如果协议不是HTTP的，就无能为力了；
*   小组童鞋虽然开发了联调系统，可以自动切换项目的网络代理，但是如果项目在早期阶段，是无法使用的；
*   用路由等方案组网做小范围调试(等待基建共用调试网络)，也会遇到要一次次设置DNS配置...

## 解决方案

下面是根据`Alfred switch DNS`的脚本改的一个版本，扩展了设置搜索域的功能，稍稍改变了编码风格。 真实使用记得修改`DNS_PROFILES`中的`YOUR DNS`, : )

```bash
#!/bin/bash

#
# Description: switch DNS profile quickly
# Author: kodango <dangoakachan@foxmail.com>, soulteary <soulteary@gmail.com>
#

# DNS profiles
# profile name::dns servers
DNS_PROFILES=$(cat <<EOF
Default DNS::empty::empty
Alibaba Public DNS::223.5.5.5 223.6.6.6::empty
V2EX Public DNS::199.91.73.222 178.79.131.110::empty
114 Public DNS::114.114.114.114 114.114.115.115::empty
Google Public DNS::8.8.8.8 8.8.4.4::empty
OpenerDNS::42.120.21.30::empty
YOUR DNS::192.168.123.123 192.168.234.234::YOUR.DNS.SEARCH.DOMAIN
EOF
)

# Swith to matched dns profile
function switch_dns()
{
    local MSG_ERROR_INPUT="没有定义过的DNS服务"
    local MSG_SWICH_TO="切换 DNS 服务为"
    local profile=$(echo "$DNS_PROFILES" | grep -iE "$1")
    local name server domain

    name=$(echo "$profile" | awk -F:: '{print $1;exit}')
    server=$(echo "$profile" | awk -F:: '{print $2;exit}')
    domain=$(echo "$profile" | awk -F:: '{print $3;exit}')

    if [ -z "$name" ]; then
        echo "$MSG_ERROR_INPUT: '$1'"
        return
    fi

    local current_dev=$(netstat -rn | awk '/default/{print $NF}')
    local current_dev_name=$(networksetup -listnetworkserviceorder | awk "/$current_dev/{print \$3}" | tr -d ', ')

    sudo networksetup -setsearchdomains "$current_dev_name" $domain &&
    sudo networksetup -setdnsservers "$current_dev_name" $server

    dscacheutil -flushcache

    echo "$MSG_SWICH_TO '$name'"
}

switch_dns "{query}"
```

