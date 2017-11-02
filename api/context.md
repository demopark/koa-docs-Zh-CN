# 上下文(Context)

Koa Context 将 node 的 `request` 和 `response` 对象封装到单个对象中，为编写 Web 应用程序和 API 提供了许多有用的方法。
这些操作在 HTTP 服务器开发中频繁使用，它们被添加到此级别而不是更高级别的框架，这将强制中间件重新实现此通用功能。

_每个_ 请求都将创建一个 `Context`，并在中间件中作为接收器引用，或者 `ctx` 标识符，如以下代码片段所示：

```js
app.use(async ctx => {
  ctx; // 这是 Context
  ctx.request; // 这是 koa Request
  ctx.response; // 这是 koa Response
});
```

为方便起见许多上下文的访问器和方法直接委托给它们的 `ctx.request `或 `ctx.response` ，不然的话它们是相同的。
例如 `ctx.type` 和 `ctx.length` 委托给 `response` 对象，`ctx.path` 和 `ctx.method` 委托给 `request`。

## API

  `Context` 具体方法和访问器.

### ctx.req

  Node 的 `request` 对象.

### ctx.res

  Node 的 `response` 对象.

  绕过 Koa 的 response 处理是 __不被支持的__. 应避免使用以下 node 属性：

- `res.statusCode`
- `res.writeHead()`
- `res.write()`
- `res.end()`

### ctx.request

  koa 的 `Request` 对象.

### ctx.response

  koa 的 `Response` 对象.

### ctx.state

推荐的命名空间，用于通过中间件传递信息和你的前端视图。

```js
ctx.state.user = await User.find(id);
```

### ctx.app

  应用程序实例引用

### ctx.cookies.get(name, [options])

   通过 `options` 获取 cookie `name`:

 - `signed` 所请求的cookie应该被签名

koa 使用 [cookies](https://github.com/jed/cookies) 模块，其中只需传递参数。

### ctx.cookies.set(name, value, [options])

  通过 `options` 设置 cookie `name` 的 `value` :

 - `maxAge` 一个数字表示从 Date.now() 得到的毫秒数
 - `signed` cookie 签名值
 - `expires` cookie 过期的 `Date`
 - `path` cookie 路径, 默认是`'/'`
 - `domain` cookie 域名
 - `secure` 安全 cookie
 - `httpOnly` 服务器可访问 cookie,  默认是 __true__
 - `overwrite` 一个布尔值，表示是否覆盖以前设置的同名的 cookie (默认是 __false__). 如果是 true, 在同一个请求中设置相同名称的所有 Cookie（不管路径或域）是否在设置此Cookie 时从 Set-Cookie 标头中过滤掉。

koa 使用传递简单参数的 [cookies](https://github.com/jed/cookies) 模块。

### ctx.throw([status], [msg], [properties])

Helper 方法抛出一个 `.status` 属性默认为 `500` 的错误，这将允许 Koa 做出适当地响应。

允许以下组合：

```js
ctx.throw(400);
ctx.throw(400, 'name required');
ctx.throw(400, 'name required', { user: user });
```

  例如 `ctx.throw(400, 'name required')` 等效于:

```js
const err = new Error('name required');
err.status = 400;
err.expose = true;
throw err;
```

请注意，这些是用户级错误，并用 `err.expose` 标记，这意味着消息适用于客户端响应，这通常不是错误消息的内容，因为您不想泄漏故障详细信息。
 
你可以根据需要将 `properties` 对象传递到错误中，对于装载上传给请求者的机器友好的错误是有用的。这用于修饰其人机友好型错误并向上游的请求者报告非常有用。

```js
ctx.throw(401, 'access_denied', { user: user });
```

koa 使用 [http-errors](https://github.com/jshttp/http-errors) 来创建错误。

### ctx.assert(value, [status], [msg], [properties])

当 `!value` 时，Helper 方法抛出类似于 `.throw()` 的错误。这与 node 的 [assert()](http://nodejs.org/api/assert.html) 方法类似.

```js
ctx.assert(ctx.state.user, 401, 'User not found. Please login!');
```

koa 使用 [http-assert](https://github.com/jshttp/http-assert) 作为断言。

### ctx.respond

为了绕过 Koa 的内置 response 处理，你可以显式设置 `ctx.respond = false;`。 如果您想要写入原始的 `res` 对象而不是让 Koa 处理你的 response，请使用此参数。

请注意，Koa _不_ 支持使用此功能。这可能会破坏 Koa 中间件和 Koa 本身的预期功能。使用这个属性被认为是一个 hack，只是便于那些希望在 Koa 中使用传统的 `fn(req, res)` 功能和中间件的人。

## Request 别名

以下访问器和 [Request](request.md) 别名等效：

  - `ctx.header`
  - `ctx.headers`
  - `ctx.method`
  - `ctx.method=`
  - `ctx.url`
  - `ctx.url=`
  - `ctx.originalUrl`
  - `ctx.origin`
  - `ctx.href`
  - `ctx.path`
  - `ctx.path=`
  - `ctx.query`
  - `ctx.query=`
  - `ctx.querystring`
  - `ctx.querystring=`
  - `ctx.host`
  - `ctx.hostname`
  - `ctx.fresh`
  - `ctx.stale`
  - `ctx.socket`
  - `ctx.protocol`
  - `ctx.secure`
  - `ctx.ip`
  - `ctx.ips`
  - `ctx.subdomains`
  - `ctx.is()`
  - `ctx.accepts()`
  - `ctx.acceptsEncodings()`
  - `ctx.acceptsCharsets()`
  - `ctx.acceptsLanguages()`
  - `ctx.get()`

## Response 别名

以下访问器和 [Response](response.md) 别名等效：

  - `ctx.body`
  - `ctx.body=`
  - `ctx.status`
  - `ctx.status=`
  - `ctx.message`
  - `ctx.message=`
  - `ctx.length=`
  - `ctx.length`
  - `ctx.type=`
  - `ctx.type`
  - `ctx.headerSent`
  - `ctx.redirect()`
  - `ctx.attachment()`
  - `ctx.set()`
  - `ctx.append()`
  - `ctx.remove()`
  - `ctx.lastModified=`
  - `ctx.etag=`
