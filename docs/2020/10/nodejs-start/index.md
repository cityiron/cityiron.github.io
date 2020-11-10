# 一拳搞定 Node.js | 如何运行一个 node 项目

本文介绍如何在本地启动 nodejs 项目。

<!--more-->

## 一、前言

最近要搭建自动化运维平台，前后端的生杀大权都随自己定，前端的技术选型了蚂蚁的 `ant design pro`，于是乎把折腾过的 `node` 和 `react` 的相关知识记录下。

注意：
- 本文需要读者具备基础的编程能力以及对计算机网络的基本了解，一些常用术语限于篇幅不再展开。
- 本文基于 mac 系统开发，如果 win 下有不一致的情况请自行对照相关命令。

## 二、准备环境

### 2.1、安装 Node.js

Node.js 发布版本分为 Current 和 LTS 两类，前者每 6 个月迭代一个大版本，快速提供新增功能和问题修复，后者以 30 个月为一个大周期，为生产环境提供稳定依赖。当前 Node.js 最新 LTS 版本为 12.18.2，本文以此版本作为运行环境，可以在官网下载并安装。

安装完成后，在命令行输入 `node --version` 查看输出是否为 `v12.8.2`，如果一致，那么安装成功。

另外，有兴趣的读者可以尝试通过 `nvm / nvm-windows` 管理多个 Node.js 版本，此处不是本文重点不再展开。

我本机的版本：

```bash
$ node -v
v12.18.2
```

## 三、创建 node 项目

### 3.1 初始化工程

#### 3.1.1 创建并进入项目目录

```bash
mkdir node-sample && cd node-sample
```

#### 3.1.2 初始化 package.json

```bash
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (sample) demo
version: (1.0.0) 0.1.0
description: demo node
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /Users/tc/Documents/workspace_2020/node-sample/package.json:

{
  "name": "demo",
  "version": "0.1.0",
  "description": "demo node",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes) yes
```

查看生成的文件或目录

```bash
$ ls
node_modules            package-lock.json       package.json
```

#### 3.1.3 使用 express

安装 `[express](https://expressjs.com/)`

```bash
 npm install express
```

创建目录和文件

```bash
mkdir src
touch src/server.js
```

编写 `server.js`

```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

#### 3.1.4 启动服务

启动

```bash
$ node src/server.js
Example app listening at http://localhost:3000
```

请求

{{< figure src="/2020/10/nodejs-start/request.jpg" title="请求" >}}

停止服务

```bash
control + c
```

端口是否在使用

```bash
$ lsof -i:3000
```

### 3.2、静态资源服务器

#### 3.2.1 创建目录和文件

创建 public 文件夹

```bash
mkdir src
```

创建 HTML 文件

```bash
touch src/index.html
```

查看目录

```bash
$ tree -L 1
.
├── index.js
├── node_modules
├── package-lock.json
├── package.json
├── public
└── src
```

#### 3.2.2 编写 index.html

```HTML
<!doctype html>
  <html>
    <head>
      <meta charset="utf-8" />
    </head>
    <body>
      <h1>It works!</h1>
    </body>
  </html>
```

#### 3.2.3 编写 server.js

```js
const express = require('express')
const { resolve } = require('path');
const { promisify } = require('util');

const app = express()
const port = parseInt(process.env.PORT || '3000');
const publicDir = resolve('public');

async function bootstrap() {
  app.use(express.static(publicDir))
  await promisify(app.listen.bind(app, port))();
  console.log(`> Started on port http://localhost:${port}`);
}

bootstrap();
```

#### 3.2.4 启动服务

启动

```bash
npm start

> demo@0.1.0 start /Users/tc/Documents/workspace_2020/node-sample
> node src/server.js

> Started on port http://localhost:3000
```

请求

{{< figure src="/2020/10/nodejs-start/request2.jpg" title="请求" >}}

## 四、碰到问题

### 4.1 端口被占用

配置好 package.json 的 start script 后，启动项目。

```bash
npm start

> demo@0.1.0 start /Users/tc/Documents/workspace_2020/node-sample
> node src/server.js

events.js:292
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE: address already in use :::3000
    at Server.setupListenHandle [as _listen2] (net.js:1313:16)
    at listenInCluster (net.js:1361:12)
    at Server.listen (net.js:1447:7)
    at Function.listen (/Users/tc/Documents/workspace_2020/node-sample/node_modules/express/lib/application.js:618:24)
    at internal/util.js:297:30
    at new Promise (<anonymous>)
    at bound listen (internal/util.js:296:12)
    at bootstrap (/Users/tc/Documents/workspace_2020/node-sample/src/server.js:15:46)
    at Object.<anonymous> (/Users/tc/Documents/workspace_2020/node-sample/src/server.js:19:1)
    at Module._compile (internal/modules/cjs/loader.js:1138:30)
Emitted 'error' event on Server instance at:
    at emitErrorNT (net.js:1340:8)
    at processTicksAndRejections (internal/process/task_queues.js:84:21) {
  code: 'EADDRINUSE',
  errno: 'EADDRINUSE',
  syscall: 'listen',
  address: '::',
  port: 3000
}
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! demo@0.1.0 start: `node src/server.js`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the demo@0.1.0 start script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/tc/.npm/_logs/2020-11-10T02_13_36_826Z-debug.log
```

可以看到错误日志的日志文件位置和错误信息，地址已经被占用。

## 五、参考

https://expressjs.com/en/starter/hello-world.html
https://segmentfault.com/a/1190000023311123

