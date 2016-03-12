# Ubuntu 18.04 基础系统配置

之前迁移 **GitLab** 的时候[有提过](https://soulteary.com/2018/09/27/migrate-your-gitlab.html)，我在公有云上使用了 **Ubuntu 18.04**，而家里的服务器一律还是 **16.04**。

随着时间的推移，我现在需要管理和折腾的机器越来越多，除了公司有要求使用同一的系统版本外，为了减少维护成本，我已然将接触的机器全部更新至 **18.04**。

本篇内容将相对详细又不失简单的介绍如何配置最基础的系统环境。

## 升级老版本到最新版本

跨大版本升级很简单，只需要一条命令：

```bash
do-release-upgrade
```

然后根据自己情况进行选择，一般情况，一路 Next 就好了。

不过如果你已经是最新的版本了，只想升级小版本，发现刚刚这条命令执行后没有效果。

那么需要将 `/etc/update-manager/release-upgrades` 里的 `Prompt=lts` 改为 `Prompt=normal` 后，再执行命令。

接着讲讲新系统如何配置吧。

## 配置基础环境

拿到新系统，该做一些什么事情呢。

### 配置系统源

第一件事推荐修改镜像源，根据机器的地域进行调整，比如在国内，可以选择阿里云的源。

```bash
# 编辑源文件
sudo vim /etc/apt/sources.list
# 在VIM编辑器内替换默认源为阿里云
:0,$ s/archive.ubuntu.com/mirrors.aliyun.com/
# 保存源文件
:wq
```

### 执行系统更新

接着执行系统更新，并更新已经安装的软件。

```bash
apt update && apt upgrade -y
```

### 安装语言包

如果你想在系统上愉快的查看中文信息，而不是乱码或者问号，需要安装下面的两个语言包。

```bash
apt install language-pack-zh-hant language-pack-zh-hans -y
```

### 配置时区

当然，也不要忘记配置系统时区，尤其是现在流行将系统时区配置挂载到容器中。

```bash
dpkg-reconfigure tzdata
```

### 安装常用软件

安装一些常用软件。

```bash
apt install git zsh wget curl unzip vim -y
```

如果经常登录系统执行命令，可以考虑安装 `ZSH`。

```bash
curl -L http://install.ohmyz.sh | sh
```

### 配置免登陆

使用 RSA Key 进行系统登录。

```bash
ssh-copy-id rsa-key.pub HOST_IP
```

修改配置 `vim /etc/ssh/sshd_config` 文件，禁用密码登录，以及尽可能避免使用 **root** 用户直接登录系统。

```bash
PermitRootLogin no
PasswordAuthentication no
```

最后重启 `ssh` 服务即可。

```bash
sudo service ssh restart
```

### 安装容器环境

安装容器环境。

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt install -y docker-ce
```

如果你的系统在阿里云，只有内网访问权限，缺乏公网访问能力，那么可以使用下面的源进行容器安装。

```bash
deb [arch=amd64] https://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu bionic stable
```

根据自己的情况，选择是否锁定容器环境，避免升级带来不确定性。

```bash
apt-mark docker-ce
```

根据自己情况，选择是否安装 `Compose`。

```bash
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

配置容器镜像源。

```bash
mkdir -p /etc/docker && touch /etc/docker/daemon.json

cat <<EOF > /etc/docker/daemon.json
{
    "registry-mirrors": [
        "http://你的镜像地址"
    ]
}
EOF

service docker restart
```

### 处理数据盘

系统默认不会自动格式化以及挂载磁盘，需要手动操作一下。

先使用下面的命令，查看你的磁盘信息。

```bash
fdisk -l
```

然后针对具体的磁盘进行分区操作，比如 `vdb`。

```bash
fdisk -u /dev/vdb
```

交互式输入 ：`p-> n-> p-> 回车-> 回车-> 回车-> w`

然后格式化磁盘。

```bash
mkfs.ext4 /dev/vdb1
```

将磁盘写入系统分区配置表中。

```bash
echo /dev/vdb1 /data ext4 defaults 0 0 >> /etc/fstab
```

接着重启系统，或者使用 `mount -a` 让刚刚的操作生效。

## 最后

Ubuntu 已经不知不觉的陪伴了我[一个生肖轮回](https://soulteary.com/2007/09/30/ubuntu.html)，从最开始的简陋至极到现在的衍生版百家争鸣，从单纯的偶尔用用到现在工作中必不可少，还是很感慨的。

希望未来的 Ubuntu 可以更好，在 IOT、 Cloud 领域越来越强。
