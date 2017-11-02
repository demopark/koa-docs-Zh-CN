# 安装

Koa 依赖 __node v7.6.0__ 或 ES2015及更高版本和 async 方法支持.

你可以使用自己喜欢的版本管理器快速安装支持的 node 版本：

```bash
$ nvm install 7
$ npm i koa
$ node my-koa-app.js
```

## 使用 Babel 实现 Async 方法

要在 node < 7.6 版本的 Koa 中使用 `async` 方法, 我们推荐使用 [babel's require hook](http://babeljs.io/docs/usage/babel-register/).

```js
require('babel-register');
// 应用的其余 require 需要被放到 hook 后面
const app = require('./app');
```

要解析和编译 async 方法, 你至少应该有 [transform-async-to-generator](http://babeljs.io/docs/plugins/transform-async-to-generator/)
或 [transform-async-to-module-method](http://babeljs.io/docs/plugins/transform-async-to-module-method/) 插件.

例如, 在你的 `.babelrc` 文件中, 你应该有:

```json
{
  "plugins": ["transform-async-to-generator"]
}
```

你也可以用 [env preset](http://babeljs.io/docs/plugins/preset-env/) 的 target 参数 `"node": "current"` 替代.

# 应用程序

Koa 应用程序是一个包含一组中间件函数的对象，它是按照类似堆栈的方式组织和执行的。
Koa 类似于你可能遇到过的许多其他中间件系统，例如 Ruby 的 Rack ，Connect 等，然而，一个关键的设计点是在其低级中间件层中提供高级“语法糖”。 这提高了互操作性，稳健性，并使书写中间件更加愉快。

这包括诸如内容协商，缓存清理，代理支持和重定向等常见任务的方法。 尽管提供了相当多的有用的方法 Koa 仍保持了一个很小的体积，因为没有捆绑中间件。

必修的 hello world 应用:

```js
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

## 级联

Koa 中间件以更传统的方式级联，您可能习惯使用类似的工具 - 之前难以让用户友好地使用 node 的回调。然而，使用 async 功能，我们可以实现 “真实” 的中间件。对比 Connect 的实现，通过一系列功能直接传递控制，直到一个返回，Koa 调用“下游”，然后控制流回“上游”。

下面以 “Hello World” 的响应作为示例，首先请求流通过 `x-response-time` 和 `logging` 中间件来请求何时开始，然后继续移交控制给 `response` 中间件。当一个中间件调用 `next()`  则该函数暂停并将控制传递给定义的下一个中间件。当在下游没有更多的中间件执行后，堆栈将展开并且每个中间件恢复执行其上游行为。

```js
const Koa = require('koa');
const app = new Koa();

// x-response-time

app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set('X-Response-Time', `${ms}ms`);
});

// logger

app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}`);
});

// response

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

## 设置

应用程序设置是 `app` 实例上的属性，目前支持如下：

  - `app.env` 默认是 __NODE_ENV__ 或 "development"
  - `app.proxy` 当真正的代理头字段将被信任时
  - `app.subdomainOffset` 对于要忽略的 `.subdomains` 偏移[2]

## app.listen(...)

  A Koa application is not a 1-to-1 representation of an HTTP server.
  One or more Koa applications may be mounted together to form larger
  applications with a single HTTP server.

  Create and return an HTTP server, passing the given arguments to
  `Server#listen()`. These arguments are documented on [nodejs.org](http://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback). The following is a useless Koa application bound to port `3000`:

```js
const Koa = require('koa');
const app = new Koa();
app.listen(3000);
```

  The `app.listen(...)` method is simply sugar for the following:

```js
const http = require('http');
const Koa = require('koa');
const app = new Koa();
http.createServer(app.callback()).listen(3000);
```

  This means you can spin up the same application as both HTTP and HTTPS
  or on multiple addresses:

```js
const http = require('http');
const https = require('https');
const Koa = require('koa');
const app = new Koa();
http.createServer(app.callback()).listen(3000);
https.createServer(app.callback()).listen(3001);
```

## app.callback()

  Return a callback function suitable for the `http.createServer()`
  method to handle a request.
  You may also use this callback function to mount your koa app in a
  Connect/Express app.

## app.use(function)

  Add the given middleware function to this application. See [Middleware](https://github.com/koajs/koa/wiki#middleware) for
  more information.

## app.keys=

 Set signed cookie keys.

 These are passed to [KeyGrip](https://github.com/jed/keygrip),
 however you may also pass your own `KeyGrip` instance. For
 example the following are acceptable:

```js
app.keys = ['im a newer secret', 'i like turtle'];
app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256');
```

  These keys may be rotated and are used when signing cookies
  with the `{ signed: true }` option:

```js
ctx.cookies.set('name', 'tobi', { signed: true });
```

## app.context

  `app.context` is the prototype from which `ctx` is created from.
  You may add additional properties to `ctx` by editing `app.context`.
  This is useful for adding properties or methods to `ctx` to be used across your entire app,
  which may be more performant (no middleware) and/or easier (fewer `require()`s)
  at the expense of relying more on `ctx`, which could be considered an anti-pattern.

  For example, to add a reference to your database from `ctx`:

```js
app.context.db = db();

app.use(async ctx => {
  console.log(ctx.db);
});
```

Note:

- Many properties on `ctx` are defined using getters, setters, and `Object.defineProperty()`. You can only edit these properties (not recommended) by using `Object.defineProperty()` on `app.context`. See https://github.com/koajs/koa/issues/652.
- Mounted apps currently use its parent's `ctx` and settings. Thus, mounted apps are really just groups of middleware.

## 错误处理

默认情况下，将所有错误输出到 stderr，除非 `app.silent` 为 `true`。
当 `err.status` 是 `404` 或 `err.expose` 是 `true` 时默认错误处理程序也不会输出错误。
要执行自定义错误处理逻辑，如集中式日志记录，您可以添加一个 “error” 事件侦听器：

```js
app.on('error', err => {
  log.error('server error', err)
});
```

如果 req/res 期间出现错误，并且 _无法_ 响应客户端，`Context`实例仍然被传递：

```js
app.on('error', (err, ctx) => {
  log.error('server error', err, ctx)
});
```

当发生错误 _并且_ 仍然可以响应客户端时，
也没有数据已经写入插座，
Koa将对500个“内部服务器错误”进行适当的响应。
在任一情况下，为了记录目的，都会发出应用级“错误”。

When an error occurs _and_ it is still possible to respond to the client, aka no data has been written to the socket, Koa will respond appropriately with a 500 "Internal Server Error". In either case an app-level "error" is emitted for logging purposes.
