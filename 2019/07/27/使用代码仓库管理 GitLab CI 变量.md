# 使用代码仓库管理 GitLab CI 变量

随着越来越多的项目用上了自动化构建，我们不得不在项目中一遍遍的配置持续集成中使用的环境变量，十几个项目规模还好说，但是项目成百上千后，维护不同项目/不同项目分组变量的工作量也变的大了起来。

在大公司中，如果有团队维护基础技术设施，我们可以使用类似可配置的构建平台/应用配置中心等方案来解决这个问题。但是这类方案对于中小规模的团队或者个人开发者来说却不是那么友好、甚至可以说投入成本过高。

本文将介绍如何使用代码仓库管理项目/项目组变量，低成本解决项目在CI/CD过程中环境变量维护的问题。

## 写在前面

使用代码仓库管理应用文件配置你一定听说过或者用过，但是使用代码仓库管理环境变量，你或许就不一定用过了。

 在聊具体方案之前，我们先了解下这两种配置的异同。它们的共同点是，都储存了项目构建/运行所需要的必要信息。那么他们主要的不同点是什么呢？

![CI/CD 变量和文件配置差异](https://attachment.soulteary.com/2019/07/27/compare.jpg)

- 项目 CI/CD 变量：存放于 GitLab 项目/项目组设置页面中变量配置中的字段、在 CI/CD 过程中使用。
- 项目配置文件：使用某种具体格式书写，存放于项目仓库某个位置，例如：`./config/app.json` 或者 `./app.ini`等。在项目运行后使用。

简单来说就是：存放位置不同、使用时机不同。

我们都知道显式声明（Explicit declaration）对于维护性的利好，那么如果我们能够把变量也使用配置的方式来管理维护，问题就解决啦，比如像下面这样使用：

![新的变量使用流程](https://attachment.soulteary.com/2019/07/27/workflow.jpg)

1. 读取存放在文件中的变量信息
2. 解析每一条配置
3. 写入 GitLab CI 变量配置

## 依赖条件

[官方文档](https://docs.gitlab.com/12.1/ee/api/group_level_variables.html) 中有提到 `Group-level Variables API`，可以对项目组的变量进行“CRUD”。（操作 [Project-level Variables](https://docs.gitlab.com/12.1/ee/api/project_level_variables.html) 同理）

使用有项目访问权限的账号，打开 `https://gitlab.domain.com/profile/personal_access_tokens` ，勾选 `API` 和 `read_repository` 权限，然后生成一枚类似 `x6oeuvvfsoultearyZ2o` 的 Access Token。

![打开分组 CI 配置菜单](https://attachment.soulteary.com/2019/07/27/group-example.png)

有了这枚 Token ，我们就能模拟用户对 GitLab 进行变量配置操作了。

![获取 Access-Token](https://attachment.soulteary.com/2019/07/27/generate-access-token.png)

## 编写程序

相比较官方实例中的 `Bash` 语句，`Node.js` 等高级语言编写的脚本能在完成相同事情的时候，行数更短，比如下面的60来行程序可以解决这个问题：根据配置文件中定义的变量内容，设置多个项目或项目组、以及指定ID的项目或者项目组的变量配置。

```js
const https = require('https');
const axios = require('axios');
const instance = axios.create({ httpsAgent: new https.Agent({ rejectUnauthorized: false }) });

/**
 * settings @see `setting.example.json`
 */
const settings = require('./settings.json');
const { baseHost, token, groupIds, projectIds, groupVars, projectVars } = settings;
const options = { headers: { 'PRIVATE-TOKEN': token } };

function combine(varList) {
    if (!varList) return {};
    const { privateVars, publicVars } = varList;
    return Object.keys(publicVars).map((label) => { return { label, value: publicVars[label], protected: false } })
        .concat(Object.keys(privateVars).map((label) => { return { label, value: privateVars[label], protected: true } }))
        .reduce((r, i) => { r[i.label] = i; return r; }, {});
}

function update(itemIds, varsData, type) {
    type = type.slice(-1) === 's' ? type.slice(0, -1) : type;
    const apiType = `${type}s`;
    itemIds.forEach(async (itemId) => {
        try {
            var { data: variablesExists } = await instance.get(`${baseHost}/${apiType}/${itemId}/variables`, options);
        } catch (error) {
            return console.log(error);
        }

        const variablesKeyExists = variablesExists.map((variable) => variable.key);
        const newVariableKeys = Object.keys(varsData).filter((key) => !variablesKeyExists.includes(key));

        variablesKeyExists.forEach(async (key) => {
            if (!varsData[key]) return;
            const { value, protected } = varsData[key];
            try {
                await instance.put(`${baseHost}/${apiType}/${itemId}/variables/${key}`, `value=${value}&protected=${protected}`, options);
            } catch (error) {
                return console.log(error);
            }
            console.log(`Update #${itemId} ${type}: [${key}]`);
        });

        newVariableKeys.forEach(async (key) => {
            if (!varsData[key]) return;
            const { value, protected } = varsData[key];
            try {
                await instance.post(`${baseHost}/${apiType}/${itemId}/variables`, `key=${key}&value=${value}&protected=${protected}`, options);
            } catch (error) {
                return console.log(error);
            }
            console.log(`Create #${itemId} ${apiType}: [${key}]`);
        });

        if (settings[`${type}Vars:${itemId}`]) {
            const itemData = combine(settings[`${type}Vars:${itemId}`]);
            delete settings[`${type}Vars:${itemId}`];
            update([itemId], itemData, type);
        }
    });
}

const groupsVarsList = combine(groupVars);
const projectVarsList = combine(projectVars);

update(groupIds, groupsVarsList, 'group');
update(projectIds, projectVarsList, 'project');
```

想要使用这段脚本，需要创建一个配置文件 `setting.json`，将上面获取的 Token、你的 GitLab 仓库地址，以及你想配置的项目或项目组 id 放进去，一个相对完整的例子是下面这样。

```js
{
    "baseHost": "https://gitlab.lab.com/api/v4",
    "token": "H7hHmdCnryy7UCeFB7tH",
    "groupIds": [
        5
    ],
    "groupVars": {
        "publicVars": {
            "VAR_PUB": 1024
        },
        "privateVars": {
            "VAR_HIDE": 1024,
            "VAR_HIDE2": 1024
        }
    },
    "projectIds": [
        774,
        775
    ],
    "projectVars": {
        "publicVars": {
            "VAR_PUB": 1024
        },
        "privateVars": {
            "VAR_HIDE": 1024,
            "VAR_HIDE2": 1024
        }
    },
    "projectVars:775": {
        "publicVars": {
            "VAR_775_PUB": 2048
        },
        "privateVars": {
            "VAR_775_HIDE": 2048,
            "VAR_775_HIDE2": 2048
        }
    }
}
```

这里除了 `baseHost`、`token` 字段外，其他的配置都是选配的，包括二级字段 `publicVars`、`privateVars`，所以如果你只是想配置两个项目组，的公开变量，配置会简短不少。

```js
{
    "baseHost": "https://gitlab.lab.com/api/v4",
    "token": "H7hHmdCnryy7UCeFB7tH",
    "groupIds": [
        5, 6
    ],
    "groupVars": {
        "publicVars": {
            "VAR_PUB": 1024
        }
    }
}
```

考虑到不是每个同学都熟悉 JavaScript，我这里把它封装成了容器镜像。

## 构建工具容器镜像

镜像文件十分简单，基于 Node 官方镜像，10 行以内指令解决问题。

```bash
FROM node:12.7.0-alpine

LABEL MAINTAINER="soulteary"

COPY ./src /app

WORKDIR /app

RUN npm i --production

VOLUME [ "/app/settings.json" ]

ENTRYPOINT [ "npm", "start" ]
```

将上面的内容保存为 `Dockerfile` ，接着使用 `docker build` 构建一个我们使用的工具镜像：

```bash
docker build -t soulteary/gitlab-variable-helper .
```

当然，你也可以直接使用我在 DockerHub 上提供的公开镜像：[soulteary/gitlab-variable-helper ](https://cloud.docker.com/repository/docker/soulteary/gitlab-variable-helper) 。
## 如何使用

在准备好你的配置文件 `settings.json` 后，你可以在本地环境或者服务器、或是 GitLab Runner 中执行这个工具。

执行方法除了安装好 Node.js 后执行 `node .` 或 `npm start` 外，还可以选择使用容器来执行：

```bash
docker run --rm -v `pwd`/settings.json:/app/settings.json soulteary/gitlab-variable-helper:1.0.0
```

当然，我更推荐的是使用 `compose` 文件进行容器执行，因为看起来会更加的清晰。

```yaml
version: '3'

services:

  updater:
    image: docker.lab.com/gitlab-group-variable-update:1.0.0
    volumes:
     - ./config:/app/config
```

将上面的文件保存为 `docker-compose.yml` 后，我们可以再编写一个 `.gitlab-ci.yml` ，让变量配置变的“自动”起来：

```yaml
stages:
  - deploy

update:
  stage: deploy
  script:
    - docker-compose down && docker-compose up
    # 或者
    # - docker run --rm -v `pwd`/settings.json:/app/settings.json soulteary/gitlab-variable-helper:1.0.0
```

如果你CI配置正确，每当你调整 `settings.json`内容，并使用 `git push` 将内容提交到 GitLab 后，都将会看到类似下面的日志输出。

![变量配置成功](https://attachment.soulteary.com/2019/07/27/auto.png)

```TeXT
docker-compose down && docker-compose up
Creating gitlab-group-update-test_updater_1 ... done
Attaching to gitlab-group-update-test_updater_1
updater_1  |
updater_1  | > gitlab-group-variable-helper@1.0.0 start /app
updater_1  | > node .
updater_1  |
updater_1  | Create #774 project: [VAR_PUB]
updater_1  | Update #775 project: [VAR_HIDE2]
updater_1  | Create #5 group: [VAR_PUB]
updater_1  | Update #5 group: [VAR_HIDE]
updater_1  | Create #775 project: [VAR_HIDE]
updater_1  | Create #774 project: [VAR_HIDE]
updater_1  | Update #5 group: [VAR_HIDE2]
updater_1  | Update #775 project: [VAR_PUB]
updater_1  | Update #774 project: [VAR_HIDE2]
updater_1  | Update #775 project: [VAR_775_HIDE2]
updater_1  | Update #775 project: [VAR_775_PUB]
updater_1  | Update #775 project: [VAR_775_HIDE]
gitlab-group-update-test_updater_1 exited with code 0
```

![设置变量完毕](https://attachment.soulteary.com/2019/07/27/setup.png)

打开项目/项目组页面，可以看到变量已经被成功设置完毕了。

完整项目，我已经提交到 GitHub 了：[https://github.com/soulteary/gitlab-variable-helper](https://github.com/soulteary/gitlab-variable-helper)，感兴趣的同学可以自取。

## 最后

懒是程序员的美德，为了能变懒去折腾也是。

—EOF
