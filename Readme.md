<img src="logo.png" alt="用于 nodejs 的 koa 中间件框架"/>

  [![gitter][gitter-image]][gitter-url]
  [![NPM version][npm-image]][npm-url]
  [![build status][travis-image]][travis-url]
  [![Test coverage][coveralls-image]][coveralls-url]
  [![OpenCollective Backers][backers-image]](#backers)
  [![OpenCollective Sponsors][sponsors-image]](#sponsors)
  
> 此项目同步自 [koajs](https://github.com/koajs) / [koa](https://github.com/koajs/koa) 项目中的  docs. 除特殊情况, 将保持每月一次的同步频率.

Koa 通过 node.js 实现了一个十分具有表现力的 HTTP 中间件框架，力求让 Web 应用开发和 API 使用更加地愉快。Koa 的中间件之间按照编码顺序在栈内依次执行，允许您执行操作并向下传递请求（downstream），之后过滤并逆序返回响应（upstream）。

几乎所有 HTTP 服务器通用的方法都被直接集成到 Koa 大约570行源码的代码库中。其中包括内容协商，节点不一致性的规范化，重定向等等操作。

Koa没有捆绑任何中间件。

## 安装

Koa 依赖 __node v7.6.0__ 或 ES2015及更高版本和 async 方法支持.

```
$ npm install koa
```

## Hello koa

```js
const Koa = require('koa');
const app = new Koa();

// 响应
app.use(ctx => {
  ctx.body = 'Hello Koa';
});

app.listen(3000);
```

## 入门

 - [Kick-Off-Koa](https://github.com/koajs/kick-off-koa) - 通过一系列自身指引的讲解介绍了 Koa。
 - [Workshop](https://github.com/koajs/workshop) - 通过学习 Koa 的讲解，快速领会精髓。
 - [Introduction Screencast](http://knowthen.com/episode-3-koajs-quickstart-guide/) - 关于 Koa 安装入门的介绍。


## 中间件

Koa 是一个中间件框架，可以采用两种不同的方法来实现中间件：

  * async function
  * common function

以下是使用两种不同方法实现一个日志中间件的示例：

### ___async___ functions (node v7.6+)

```js
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
```

### Common function

```js
// 中间件通常带有两个参数 (ctx, next), ctx 是一个请求的上下文（context）,
// next 是调用执行下游中间件的函数. 在代码执行完成后通过 then 方法返回一个 Promise.

app.use((ctx, next) => {
  const start = Date.now();
  return next().then(() => {
    const ms = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
  });
});
```

### Koa v1.x 中间件签名

中间件签名在 v1.x 和 v2.x 之间已经被更改. 旧的签名已经被弃用.

**旧的签名中间件支持将在 v3 中删除**

请参阅 [迁移指南](migration.md) 获取有关从 v1.x 升级并使用 v2.x 中间件的更多信息。

## 上下文, 请求和响应

每个中间件都接收一个 Koa 的 `Context` 对象，该对象封装了一个传入的 http 消息，并对该消息进行了相应的响应。 `ctx` 通常用作上下文对象的参数名称。

```js
app.use(async (ctx, next) => { await next(); });
```

Koa 提供了一个 `Request` 对象作为 `Context` 的 `request` 属性。
Koa的 `Request` 对象提供了用于处理 http 请求的方法，该请求委托给 node `http` 模块的[IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)。

这是一个检查请求客户端 xml 支持的示例。

```js
app.use(async (ctx, next) => {
  ctx.assert(ctx.request.accepts('xml'), 406);
  // 相当于:
  // if (!ctx.request.accepts('xml')) ctx.throw(406);
  await next();
});
```

Koa提供了一个 `Response` 对象作为 `Context` 的 `response` 属性。
Koa的 `Response` 对象提供了用于处理 http 响应的方法，该响应委托给 [ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse)。

Koa 对 Node 的请求和响应对象进行委托而不是扩展它们。这种模式提供了更清晰的接口，并减少了不同中间件与 Node 本身之间的冲突，并为流处理提供了更好的支持。
`IncomingMessage` 仍然可以作为 `Context` 上的 `req` 属性被直接访问，并且`ServerResponse`也可以作为`Context` 上的 `res` 属性被直接访问。

这里是一个使用 Koa 的 `Response` 对象将文件作为响应体流式传输的示例。

```js
app.use(async (ctx, next) => {
  await next();
  ctx.response.type = 'xml';
  ctx.response.body = fs.createReadStream('really_large.xml');
});
```

`Context` 对象还提供了其 `request` 和 `response` 方法的快捷方式。在前面的例子中，可以使用 `ctx.type` 而不是 `ctx.request.type`，而 `ctx.accepts` 可以用来代替 `ctx.request.accepts`。

关于 `Request`, `Response` 和 `Context` 更多详细信息, 参阅 [请求 API 参考](api/request.md),
[响应 API 参考](api/response.md) 和 [上下文 API 参考](api/context.md).

## Koa 应用程序

在执行 `new Koa()` 时创建的对象被称为 Koa 应用对象。

应用对象是带有 node http 服务的 Koa 接口，它可以处理中间件的注册，将http请求分发到中间件，进行默认错误处理，以及对上下文，请求和响应对象进行配置。

了解有关应用程序对象的更多信息请到 [应用 API 参考](api/index.md).

## 文档

 - [使用指南](guide.md)
 - [错误处理](error-handling.md)
 - [Koa 与 Express](koa-vs-express.md)
 - [常见问题](faq.md)
 - [从 Koa v1.x 迁移到 v2.x](migration.md)

 - [API 文档](api/index.md)
   - [上下文(Context)](api/context.md)
   - [请求(Request)](api/request.md)
   - [响应(Response)](api/response.md)

 - [Koa 中间件列表](https://github.com/koajs/koa/wiki)

## Babel 配置

如果你正在使用的不是 `node v7.6+`, 我们推荐你用 [`babel-preset-env`](https://github.com/babel/babel-preset-env) 配置 `babel` :

```bash
$ npm install babel-register babel-preset-env --save
```

在你入口文件配置 `babel-register`:

```js
require('babel-register');
```

还有你的 `.babelrc` 配置:

```json
{
  "presets": [
    ["env", {
      "targets": {
        "node": true
      }
    }]
  ]
}
```

## 故障排除

在 Koa 指南中查阅 [故障排除指南](troubleshooting.md) 或 [调试 Koa](guide.md#debugging-koa).

## 运行测试

```
$ npm test
```

# 赞赏支持
![赞赏支持](https://raw.githubusercontent.com/demopark/electron-api-demos-Zh_CN/master/assets/img/td.png)

[npm-image]: https://img.shields.io/npm/v/koa.svg?style=flat-square
[npm-url]: https://www.npmjs.com/package/koa
[travis-image]: https://img.shields.io/travis/koajs/koa/master.svg?style=flat-square
[travis-url]: https://travis-ci.org/koajs/koa
[coveralls-image]: https://img.shields.io/codecov/c/github/koajs/koa.svg?style=flat-square
[coveralls-url]: https://codecov.io/github/koajs/koa?branch=master
[backers-image]: https://opencollective.com/koajs/backers/badge.svg?style=flat-square
[sponsors-image]: https://opencollective.com/koajs/sponsors/badge.svg?style=flat-square
[gitter-image]: https://img.shields.io/gitter/room/koajs/koa.svg?style=flat-square
[gitter-url]: https://gitter.im/koajs/koa?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge
[#koajs]: https://webchat.freenode.net/?channels=#koajs
