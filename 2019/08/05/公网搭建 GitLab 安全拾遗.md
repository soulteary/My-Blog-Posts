# 公网搭建 GitLab 安全拾遗

在公网搭建的 GitLab 频频遇到安全挑战，然而其实只需要做一两个简单的动作，维护成本就能够大大降低，并且还能避免未被许可的内容，被搜索引擎爬虫暴露的到处都是。

本篇文章，我们就来聊聊公网搭建的 GitLab 代码仓库的安全小细节。

## 写在前面

公网搭建 GitLab ，常见的攻击面主要有：

- 运行宿主机系统部分
- 运行宿主机网络部分
- 应用 Web 程序漏洞
- 应用 SSH 漏洞

前两点可以通过 `SLB` + `VPC` 进行网络隔离，来降低被攻击风险。后两点除了保证最快跟进系统安全补丁，升级应用版本外，其实还有更好的解决方案，毕竟存放着数据的程序，每次升级都有未知的风险：

- 解决 Web 漏洞可以通过加一层 `Basic Auth` 来解决。
- 解决 SSH 攻击风险，可以通过加一个简单的日志监控程序来解决：参考之前文章中[ 监控 GitLab SSH 端口 ](https://soulteary.com/2019/04/10/gitlab-was-built-with-docker-and-traefik-part-2.html)小节。

但是加一层 `Basic Auth` 其实会对 GitLab 使用造成一些麻烦。

## 为 GitLab 添加请求验证

GitLab 程序本身并不支持 `Basic Auth`，这里需要使用一个 Web 前端软件来完成这部分的工作，比如：Nginx、Traefik。

我这里选择使用 Traefik，因为配置更简单，具体配置可以参考之前文章的“[ 添加网络请求验证 ](https://soulteary.com/2019/04/10/gitlab-was-built-with-docker-and-traefik-part-2.html)”小节。

在配置声明里添加一句话就够了，比如这样：

```xml
- "traefik.gitlab.frontend.auth.basic=soulteary:$apr1$E86fARwM$tXmggGAtCEDKqsBCSvDA3/
```

这样处理之后，所有的 HTTP 请求就都会被验证是否是合法访问啦。而其他的端口和协议则不受影响，比如开在22端口的 SSH 服务等。

### Basic Auth 到底是什么

当访问页面的时候，会展示类似下面的对话框，要求用户登陆，否则会提示 `401 Unauthorized`。

![访问页面的时候会显示一个对话框](https://attachment.soulteary.com/2019/08/05/basic-auth-dialog.png)

当爬虫/安全检测工具请求页面的时候，如果没有提交用户名和密码，获得的结果也是一样。

```TeXT
curl -I https://gitlab.domain/
HTTP/2 401
content-type: text/plain
vary: Accept-Encoding
www-authenticate: Basic realm="traefik"
content-length: 17
date: Sat, 03 Aug 2019 18:22:17 GMT
```

如果你输入了正确的用户名以及用户密码，你会发现在你的请求参数中会出现一个额外的请求参数 `authorization`。

![访问页面的时候会显示一个对话框](https://attachment.soulteary.com/2019/08/05/headers.png)

参数数值一般由两部分构成，第一部分表示加密方式（加密协议名称），第二部分则是你的身份信息（更多信息详见 [RFC 7617](https://tools.ietf.org/html/rfc7617)）。

现代浏览器一般会很智能的在你第一次正确输入之后，将身份信息记录下来，携带在后续的每一次请求中，如果是使用程序或者工具的话，则需要手动将 `authorization` 信息加入到每一个 HTTP 的请求头中。

然而 Basic Auth 挡住了外来者随意访问的同时，还挡住了 GitLab CI Runner。

![项目关联 Runner 离线](https://attachment.soulteary.com/2019/08/05/runner-offline.png)

这是怎么回事呢，我们继续往下看。

## 解救被拦住的 CI Runner

在解释为什么 CI Runner 会被 `Basic Auth` 拦住时，我们需要先了解另外一个协议规范 [RFC1738](https://tools.ietf.org/html/rfc1738) 中对于 HTTP 协议的定义：

```TeXT
//<user>:<password>@<host>:<port>/<url-path>
```

当我们使用客户端（浏览器、curl等工具）请求这类格式的地址时，部分客户端会将 `user:password` 部分转换为标准的 HTTP `Authorization` 请求头。

### 默认 CI Runner 行为

在不做任何修改时，CI 直接执行会报错，日志输出类似下面：

```TeXT
Running with gitlab-runner 12.0.2 (d0b76032)
  on ci-runner sGNUJnq6
Using Shell executor...
Running on ci-runner...
Fetching changes with git depth set to 50...
Initialized empty Git repository in /data/runner/builds/sGNUJnq6/0/config/project/.git/
Created fresh repository.
remote: 401 Unauthorized
fatal: Authentication failed for 'https://gitlab-ci-token:[MASKED]@gitlab.domain/repo.git/'
ERROR: Job failed: exit status 1
```

结合文章前面的内容，我们知道这里是缺少了 `Authorization` 请求头，那么我们尝试给请求补上这个请求头，在 CI 和 GitLab 中间搭建一台 Proxy ，让 CI 请求 GitLab 数据的时候，自动完成“认证”。

### 请求自动添加验证信息

如果使用 Nginx 搭建一个支持 GET/POST 请求的代理，核心配置如下：

```TeXT
location / {
    forward-http-post-requests-via-rewrite
    proxy_pass https://192.168.0.123;
    proxy_redirect https://192.168.0.123/ /;
    proxy_read_timeout 10s;
    proxy_set_header Host 'gitlab.domain';
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Authorization 'Basic YmFhaABCD';
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto 'https';
    proxy_http_version 1.1;
}
```

再次使用 curl 对 GitLab 进行请求，发现没有出现之前的 `401` 非验证提示：

```TeXT
curl -I https://gitlab.domain/
HTTP/2 302
cache-control: no-cache
content-type: text/html; charset=utf-8
date: Sat, 03 Aug 2019 18:45:39 GMT
location: https://gitlab.domain/users/sign_in
referrer-policy: strict-origin-when-cross-origin
referrer-policy: strict-origin-when-cross-origin
server: nginx
strict-transport-security: max-age=315360000
vary: Accept-Encoding
x-content-type-options: nosniff
x-download-options: noopen
x-frame-options: DENY
x-permitted-cross-domain-policies: none
x-request-id: Z8uahE5gN39
x-runtime: 0.012164
x-ua-compatible: IE=edge
x-xss-protection: 1; mode=block
```

打开项目配置，会发现 Runner 已经上线。

![项目关联 Runner 上线](https://attachment.soulteary.com/2019/08/05/runner-online.png)

### CI 构建依旧是失败的

继续在 GitLab Runner 运行 CI 流水线，会看到还是报错无法通过构建。

```TeXT
Running with gitlab-runner 12.2.0~beta.1803.g41d5c6ad (41d5c6ad)
  on ci-runner S8sY8o6A
Using Shell executor...
Running on ci-runner...
Fetching changes with git depth set to 50...
Reinitialized existing Git repository in /data/runner/builds/S8sY8o6A/0/project/.git/
> GitLab: The project you were looking for could not be found.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
ERROR: Job failed: exit status 1
```

还记得前面提到过的 `Authorization` 请求头和 HTTP RFC规范吗？GitLab Runner 在处理 CI 任务的时候，使用的是`https://gitlab-ci-token:[MASKED]@gitlab.domain/repo.git/` 这样的 HTTP 协议，请求中的用户名和密码和 Nginx ProxyPass 中的字段“八字不合”。

那么不使用 HTTP 协议，使用 SSH 协议或许能解决问题。

### 尝试使用 SSH 协议

可惜的是，官方并不支持 GitLab Runner 使用 SSH 协议进行仓库下载，有类似需求的用户还真不少，如果你愿意找，类似下面的 issue 还有不少：

- [Supports git clone via ssh](https://gitlab.com/gitlab-org/gitlab-runner/issues/3940)
- [Allow Configuration of how cloning is performed by runner/pipeline](https://gitlab.com/gitlab-org/gitlab-runner/issues/3055)

虽然官方没有直接提供这个功能，但是从[官方文档](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)中看到有一个 `clone_url`，查找代码发现[实现很简单](https://gitlab.com/gitlab-org/gitlab-runner/blob/v11.7.0/common/build.go#L582)，只需要稍微改造就能够满足我们的需求：

```go
// GetRemoteURL checks if the default clone URL is overwritten by the runner
// configuration option: 'CloneURL'. If it is, we use that to create the clone
// URL.
func (b *Build) GetRemoteURL() string {
	cloneURL := strings.TrimRight(b.Runner.CloneURL, "/")

	if !strings.HasPrefix(cloneURL, "http") {
		return b.GitInfo.RepoURL
	}

	variables := b.GetAllVariables()
	ciJobToken := variables.Get("CI_JOB_TOKEN")
	ciProjectPath := variables.Get("CI_PROJECT_PATH")

	splits := strings.SplitAfterN(cloneURL, "://", 2)

	return fmt.Sprintf("%sgitlab-ci-token:%s@%s/%s.git", splits[0], ciJobToken, splits[1], ciProjectPath)
}
```

改动后的代码如下：

```go
// GetRemoteURL checks if the default clone URL is overwritten by the runner
// configuration option: 'CloneURL'. If it is, we use that to create the clone
// URL.
func (b *Build) GetRemoteURL() string {
	cloneURL := strings.TrimRight(b.Runner.CloneURL, "/")

	if !strings.HasPrefix(cloneURL, "http") && !strings.HasPrefix(cloneURL, "ssh") {
		return b.GitInfo.RepoURL
	}

	variables := b.GetAllVariables()
	ciJobToken := variables.Get("CI_JOB_TOKEN")
	ciProjectPath := variables.Get("CI_PROJECT_PATH")

	splits := strings.SplitAfterN(cloneURL, "://", 2)


	if strings.HasPrefix(cloneURL, "ssh") {
        ciProjectPath = strings.TrimLeft(ciProjectPath, "/")
        return fmt.Sprintf("git@%s:/%s.git", splits[1], ciProjectPath)
    }


	return fmt.Sprintf("%sgitlab-ci-token:%s@%s/%s.git", splits[0], ciJobToken, splits[1], ciProjectPath)
}
```

如果你不想在浪费时间在折腾构建环境上，可以参考我之前写的一篇文章：[ 源码编译 GitLab Runner ](https://soulteary.com/2019/08/04/source-code-compilation-gitlab-runner.html)。

### 执行 CI 成功

再次执行 CI 任务，会发现已经能够顺利的进行啦。

```TeXT
Running on ci-runner...
Fetching changes with git depth set to 50...
Reinitialized existing Git repository in /data/runner/builds/S8sY8o6A/0/project.git/
Checking out 860587f6 as master...
Skipping Git submodules setup
Warning: Permanently added '192.168.x.x' (ECDSA) to the list of known hosts.
From gitlab.domain:project
   3957006..860587f  master     -> origin/master
Updating 3957006..860587f
Fast-forward
 .gitlab-ci.yml | 6 +++---
 README.md      | 1 +
 2 files changed, 4 insertions(+), 3 deletions(-)
Stopping service ...

Stopping service ... done
Removing service ...

Removing service ... done
Network service is external, skipping
Creating service ...

Creating service ... done
Job succeeded
```

## 最后

GitLab 先折腾到这里，或许后面有时间的会写一篇更加全面的折腾攻略。如果你对 GitLab 感兴趣，可以浏览我之前写的[相关文章](https://soulteary.com/tags/gitlab.html)。

—EOF