# 配置Ubuntu WebServer基础环境

去年年末，重新整理了一套基础环境搭建的Guide，之后在四次机器迁移中起到了蛮重要的作用（偷懒）。内容不定期更新修正，如有错误，欢迎指出。

大Debian系的Ubuntu使用起来甚是顺手。

首先根据自己的情况选择系统版本，建议使用最新的LTS。

比如：`Ubuntu 14.04 LTS` 如果安装源慢（使用国内机器/虚拟机/连接国外网络环境不佳），则替换安装源（可选）。

```bash
# 替换安装源
sudo vim /etc/apt/sources.list
# 替换资源为阿里云
: 0,$ s/us.archive.ubuntu.com/mirrors.aliyun.com/
```

如果是测试环境的虚拟机，安装好之后如果没有开sshd，需要安装 `openssh`，开启远程SSH。

```bash
# 开启远程SSH
sudo apt-get install openssh-server
```

安装完毕，检查是sshd否运行：

```bash
ps -e | grep ssh
```

为了机器和脚本的正常运行，设置系统时间。

```bash
# 设置系统时间
dpkg-reconfigure tzdata
```

更新系统，安装git、zsh、oh-my-zsh，并设置zsh为默认shell(你也可以使用你喜欢的shell)。

```bash
# 更新系统，安装必要软件，更换顺手的shell
apt-get update && apt-get upgrade
apt-get install git zsh wget curl unzip vim -y
curl -L http://install.ohmyz.sh | sh
chsh -s /bin/zsh
```

更新主机名称（如：bai）

```bash
echo "bai" > /etc/hostname
hostname -F /etc/hostname
cat /etc/hostname
```

如果运行WEB服务，且非资源代理机器，将HOSTS记录更新一下。

```bash
# 修改hosts
127.0.0.1		bai
12.34.56.78	subdomain.domain.com
```

添加用户，设置用户权限(建议运行软件单独设置用户和权限)

```bash
adduser soulteary
usermod -a -G sudo soulteary
```

设置“免”登录，如果你觉得用KEY安全系数还是低的话，尝试使用带密码的KEY。

如非自建靠谱二步验证机器，慎重考虑使用第三方二步验证软件，以免引入不必要的麻烦。

另外，如果有VNC这类的同网段管理软件，且不能附加key，请不要修改ssh_config中的禁止使用密码登录。

建议顺手改了sshd的端口，例如12345（改成你自己喜欢的高位端口号）

```bash
scp ~/.ssh/id_rsa.pub example_user@123.456.78.90:

mv id_rsa.pub .ssh/authorized_keys
chown -R example_user:example_user .ssh
chmod 700 .ssh
chmod 600 .ssh/authorized_keys

vim /etc/ssh/sshd_config

PermitRootLogin no
PasswordAuthentication no
sudo service ssh restart
```

使用PAM再次对root登录用户做判断

```bash
vim /etc/pam.d/login
# 添加内容
auth required pam_succeed_if.so user != root quiet
```

虽然用包安装的nginx比较方便维护，但是服务器一般基础设施完备后，不会轻易变动，故，推荐使用tenginx。

下面有nginx和tenginx的安装对比，如果你喜欢nginx，简单快捷的使用。

```bash
# 如果你还是喜欢nginx
apt-get install nginx
```

当然，如果你想使用更多的features。

```bash
# 如果你要安装tenginx

sudo apt-get update && sudo apt-get upgrade
apt-get install gcc g++ make -y

## 下载软件包
wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.36/pcre-8.36.tar.gz
wget http://zlib.net/zlib-1.2.8.tar.gz
wget http://www.openssl.org/source/openssl-1.0.1j.tar.gz
wget http://www.canonware.com/download/jemalloc/jemalloc-3.6.0.tar.bz2
wget http://tengine.taobao.org/download/tengine-2.1.0.tar.gz

## 安装pcre
tar zxvf pcre-8.36.tar.gz
cd pcre-8.36
./configure --prefix=/usr/local/pcre-8.36
make && make install

## 安装zlib
cd ..
tar zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure --prefix=/usr/local/zlib-1.2.8
make && make install

## 安装open-ssl
cd ..
tar zxvf openssl-1.0.1j.tar.gz
cd openssl-1.0.1j
./config --prefix=/usr/local/openssl-1.0.1j
make && make install

## 安装jemalloc
cd .. && tar jxvf jemalloc-3.6.0.tar.bz2

## 使用www-data用户和用户组安装tenginx，当然你也可以用root（不建议）
./configure --user=www-data \
--group=www-data \
--with-pcre=../pcre-8.36 \
--with-openssl=../openssl-1.0.1j/ \
--with-jemalloc=../jemalloc-3.6.0 \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_concat_module \
--with-http_spdy_module \
--with-zlib=../zlib-1.2.8
```

由于我的机器需要PHP runtime，而距离PHP7的来到还尚早，HHVM又发布了新版本，而且使用这么久，发觉性能真的是神一样的提高，故推荐HHVM。 安装hhvm，如果访问不能（虚拟机，网络），先绑定hosts。

```bash
140.211.166.134		dl.hhvm.com
```

然后进行安装

```bash
# 安装HHVM

>wget -O - http://dl.hhvm.com/conf/hhvm.gpg.key | sudo apt-key add -
echo deb http://dl.hhvm.com/ubuntu trusty main | sudo tee /etc/apt/sources.list.d/hhvm.list
apt-get update && apt-get install hhvm -y
```

如果你不需要io.js的新特性，nodejs就能满足你的话，不妨使用以下命令安装：

```bash
sudo apt-get install nodejs nodejs-legacy npm -y
```

安装mysql和redis， 因为使用supervisor管理进程，需设置redis配置daemonize:no（自己vim config就好，略）

```bash
sudo apt-get install mysql-server redis-server -y
```

安装进程管理工具

```bash
sudo apt-get install supervisor
```

配置防火墙，先创建一个打底配置。

```bash
sudo apt-get install supervisor
```

根据自己的情况选择是否开启443，如果修改过ssh的端口，顺手改掉。

```bash
*filter

# Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT -d 127.0.0.0/8 -j REJECT

# Accept all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

# Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
# -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow SSH connections
#
# The -dport number should be the same port number you set in sshd_config
#
-A INPUT -p tcp -m state --state NEW --dport 12345 -j ACCEPT

# Allow ping
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Drop all other inbound - default deny unless explicitly allowed policy
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT
```

应用防火墙规则

```bash
sudo iptables-restore < /etc/iptables.firewall.rules
sudo iptables -L
```

开机自动添加规则

```bash
sudo vim /etc/network/if-pre-up.d/firewall

#!/bin/sh
/sbin/iptables-restore < /etc/iptables.firewall.rules

sudo chmod +x /etc/network/if-pre-up.d/firewall
```

修改系统打开文件句柄数目，有人反馈开太大会导致无法登录机器，开51200吧。

```bash
## 查看当前数目
ulimit -n
## 调整数目
ulimit -SHn 51200

## 修改系统配置
vim /etc/security/limits.conf
## 添加内容
* soft nofile 51200
* hard nofile 51200

## 修改PAM
vim /etc/pam.d/common-session
## 添加内容
session required pam_limits.so

## 添加开启启动
vim /etc/profile
## 在文件末尾添加
ulimit -SHn 51200
```

修改nginx（tenginx）的打开文件句柄上限，应对突发状况。

```nginx
worker_rlimit_nofile 52100;
```

修改进程数量，不需要倍数，CPU几核心就几个线程就好。

```nginx
worker_processes 2;
```

如果你需要安装PMA，那么请手动安装，因为ubuntu下的包安装，会附带安装一堆apache相关内容，没必要。 这段忘记记录了，为了安全，请使用本地ip绑定访问pma，并开启自签名ssl，开启登录次数限制。 安装fail2ban

```bash
sudo apt-get install fail2ban
## 具体配置稍后再说，根据自己的情况需要设定不同的filter
vim /etc/fail2ban/jail.conf
```

关于服务设置，稍后详述。 部分相关内容：

*   [如果你使用PC机，且有多系统，可以使用BURG](http://www.soulteary.com/2011/10/21/ubuntu-burg.html)
*   [如果你使用windows作为servers，有一些细节可以参考](http://www.soulteary.com/2013/11/16/details-for-win-host.html)
*   [centos测试环境](http://www.soulteary.com/2013/04/13/local-test-env.html)
