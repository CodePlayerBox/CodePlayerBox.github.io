---
layout: post
title: scratch-www 代码分析
categories: [Blog]
description: 对 scratch-www 代码的初步分析
keywords: scratch, scratch-www
---


## scratch-wwww

Scratch 在 Github 上有一个 [scratch-wwww](https://github.com/LLK/scratch-www) 代码库，官方的说法是”web client for Scratch“，运行起来会发现它基本上就是 [Scratch 官网](https://scratch.mit.edu) 的前端源代码。

根据 [README](https://github.com/LLK/scratch-www/blob/develop/README.md) 中的说法，完整的 Scratch 官网在运行时至少还需要如下几个部分的支撑：

* API_HOST

API 服务器，Scratch Wiki 上目前只能找到 [Scratch API (2.0)](https://en.scratch-wiki.info/wiki/Scratch_API_(2.0))，其中的接口大多还是 2.0 时代的产物，并且有些接口已经被移除掉了。该文档中描述的部分接口在 Scratch 3.0 中还能继续使用。

* ASSET_HOST

资源服务器，用来保存像图片、声音这样的素材。

* BACKPACK_HOST

书包服务器，用来保存书包资源。

* PROJECT_HOST

项目服务器，可能是最重要的一个，所有在 Scratch 官网中创建的项目都保存在项目服务器上。

上述四个服务器目前 Scratch 官方并没有公布其源代码，所以如果想基于 scratch-www 构建一个完整的 Scratch 社区至少还需要补上这些后端才可以。

目前我们的兴趣并不在于构建一个社区，而是考虑如何扩展 Scratch 编辑器的功能，所以实际上一个简化的 scratch-www 加上一个简单的后端应该就能满足要求了。

## 基本架构

scratch-www 后端开发采用的是 [Node.js](https://nodejs.org/en/)。前端则使用了 [React](https://reactjs.org/) 框架以及 [Redux](https://redux.js.org/) 状态容器，并使用 [webpack](https://webpack.js.org/) 进行打包。

有意思的是在目前的 scratch-www 代码中居然还能找到一个 [Makefile](https://github.com/LLK/scratch-www/blob/develop/Makefile) 文件，绝对是一个超级复古的设计啊！

## 启动流程

作为一个基于 Node.js 的项目，[package.json](https://github.com/LLK/scratch-www/blob/develop/package.json) 文件中提供了启动服务器时的命令：

    "start": "make start",

继续查看 Makefile 文件可以找到：

    start:
        $(NODE) ./dev-server/index.js

在 [dev-server/index.js](https://github.com/LLK/scratch-www/blob/develop/dev-server/index.js#L9) 文件开始的位置处引入了 [src/routes.json](https://github.com/LLK/scratch-www/blob/develop/src/routes.json) 和 [src/routes-dev.json](https://github.com/LLK/scratch-www/blob/develop/src/routes-dev.json)，是在访问网站时的路由配置。

    var routes = require('../src/routes.json').concat(require('../src/routes-dev.json'))
        .filter(route => !process.env.VIEW || process.env.VIEW === route.view);

接着是创建 [Express](http://expressjs.com/) 服务器：

    var app = express();

并进行路由绑定：

    routes.forEach(route => {
        app.get(route.pattern, handler(route));
    });

随之引入 webpack 中间件：

    app.use(webpackDevMiddleware(compiler, middlewareOptions));

最后在 8333 端口上进行倾听，后端服务器启动完成：

    var port = process.env.PORT || 8333;
    app.listen(port, function () {
        process.stdout.write('Server listening on port ' + port + '\n');
        if (proxyHost) {
            process.stdout.write('Proxy host: ' + proxyHost + '\n');
        }
    });

## 路由配置

[src/routes.json](https://github.com/LLK/scratch-www/blob/develop/src/routes.json) 文件中配置了网站的主要路由，目前我们关心的主要是其中的两条：

* [splash](https://github.com/LLK/scratch-www/blob/develop/src/routes.json#L227)

访问网站首页时的路由，根据配置将对 [src/views/splash/splash.jsx](https://github.com/LLK/scratch-www/blob/develop/src/views/splash/splash.jsx) 文件进行渲染：

    {
        "name": "splash",
        "pattern": "^/?$",
        "routeAlias": "/?$",
        "view": "splash/splash",
        "title": "Imagine, Program, Share"
    }

* [projects](https://github.com/LLK/scratch-www/blob/develop/src/routes.json#L171)

创建项目时的路由，根据配置将对 [src/views/preview/preview.jsx](https://github.com/LLK/scratch-www/blob/develop/src/views/preview/preview.jsx) 文件进行渲染：

    {
        "name": "projects",
        "pattern": "^/projects(/editor|(/\\d+(/editor|/fullscreen|/embed)?)?)?/?(\\?.*)?$",
        "routeAlias": "/projects/?$",
        "view": "preview/preview",
        "title": "Scratch Project",
        "dynamicMetaTags": true
    }
