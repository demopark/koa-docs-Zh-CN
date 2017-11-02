# 请求(Request)

Koa `Request ` 对象是在 node 的 vanilla 请求对象之上的抽象，提供了诸多对 HTTP 服务器开发有用的功能。

## API

### request.header

请求标头对象。

### request.header=

设置请求标头对象。

### request.headers

请求标头对象。别名为 `request.header`.

### request.headers=

设置请求标头对象。别名为 `request.header=`.

### request.method

请求方法。

### request.method=

设置请求方法，对于实现诸如 `methodOverride()` 的中间件是有用的。

### request.length

返回以数字返回请求的 Content-Length，或 `undefined`。

### request.url

获取请求 URL.

### request.url=

设置请求 URL, 对 url 重写有用。

### request.originalUrl

获取请求原始URL。

### request.origin

获取URL的来源，包括 `protocol` 和 `host`。

```js
ctx.request.origin
// => http://example.com
```

### request.href

获取完整的请求URL，包括 `protocol`，`host` 和 `url`。

```js
ctx.request.href;
// => http://example.com/foo/bar?q=1
```

### request.path

获取请求路径名。

### request.path=

设置请求路径名，并在存在时保留查询字符串。

### request.querystring

根据 `?` 获取原始查询字符串.

### request.querystring=

设置原始查询字符串。

### request.search

使用 `?` 获取原始查询字符串。

### request.search=

设置原始查询字符串。

### request.host

获取当前主机（hostname:port）。当 `app.proxy` 是 __true__ 时支持 `X-Forwarded-Host`，否则使用 `Host`。

### request.hostname

存在时获取主机名。当 `app.proxy` 是 __true__ 时支持 `X-Forwarded-Host`，否则使用 `Host`。
  
如果主机是 IPv6, Koa 解析到
  [WHATWG URL API](https://nodejs.org/dist/latest-v8.x/docs/api/url.html#url_the_whatwg_url_api),
  *注意*  这可能会影响性能。

### request.URL

获取 WHATWG 解析的 URL 对象。

### request.type

获取请求 `Content-Type` 不含参数 "charset"。

```js
const ct = ctx.request.type;
// => "image/png"
```

### request.charset

在存在时获取请求字符集，或者 `undefined`：

```js
ctx.request.charset;
// => "utf-8"
```

### request.query

获取解析的查询字符串, 当没有查询字符串时，返回一个空对象。请注意，此 getter _不_ 支持嵌套解析。

例如 "color=blue&size=small":

```js
{
  color: 'blue',
  size: 'small'
}
```

### request.query=

将查询字符串设置为给定对象。 请注意，此 setter _不_ 支持嵌套对象。

```js
ctx.query = { next: '/login' };
```

### request.fresh

检查请求缓存是否“新鲜”，也就是内容没有改变。此方法用于 `If-None-Match` / `ETag`, 和 `If-Modified-Since` 和 `Last-Modified` 之间的缓存协商。 在设置一个或多个这些响应头后应该引用它。
  
```js
// 新鲜度检查需要状态20x或304
ctx.status = 200;
ctx.set('ETag', '123');

// 缓存是好的
if (ctx.fresh) {
  ctx.status = 304;
  return;
}

// 缓存是陈旧的
// 获取新数据
ctx.body = await db.find('something');
```

### request.stale

相反与 `request.fresh`.

### request.protocol

返回请求协议，“https” 或 “http”。当 `app.proxy` 是 __true__ 时支持  `X-Forwarded-Proto`。

### request.secure

通过 `ctx.protocol == "https"` 来检查请求是否通过 TLS 发出。

### request.ip

请求远程地址。 当 `app.proxy` 是 __true__ 时支持  `X-Forwarded-Proto`。

### request.ips

当 `X-Forwarded-For` 存在并且 `app.proxy` 被启用时，这些 ips 的数组被返回，从上游 - >下游排序。 禁用时返回一个空数组。

### request.subdomains

将子域返回为数组。

子域是应用程序主域之前主机的点分隔部分。默认情况下，应用程序的域名假定为主机的最后两个部分。这可以通过设置 `app.subdomainOffset` 来更改。

例如，如果域名为“tobi.ferrets.example.com”：

如果 `app.subdomainOffset` 未设置, `ctx.subdomains` 是 `["ferrets", "tobi"]`.
如果 `app.subdomainOffset` 是 3, `ctx.subdomains` 是 `["tobi"]`.

### request.is(types...)

检查传入请求是否包含 `Content-Type` 头字段， 并且包含任意的 mime `type`。
如果没有请求主体，返回 `null`。
如果没有内容类型，或者匹配失败，则返回 `false`。
反之则返回匹配的 content-type。 

```js
// 使用 Content-Type: text/html; charset=utf-8
ctx.is('html'); // => 'html'
ctx.is('text/html'); // => 'text/html'
ctx.is('text/*', 'text/html'); // => 'text/html'

// 当 Content-Type 是 application/json 时
ctx.is('json', 'urlencoded'); // => 'json'
ctx.is('application/json'); // => 'application/json'
ctx.is('html', 'application/*'); // => 'application/json'

ctx.is('html'); // => false
```

例如，如果要确保仅将图像发送到给定路由：

```js
if (ctx.is('image/*')) {
  // 处理
} else {
  ctx.throw(415, 'images only!');
}
```

### 内容协商

Koa的 `request` 对象包括由 [accepts](http://github.com/expressjs/accepts) 和 [negotiator](https://github.com/federomero/negotiator) 提供的有用的内容协商实体。

这些实用程序是：

- `request.accepts(types)`
- `request.acceptsEncodings(types)`
- `request.acceptsCharsets(charsets)`
- `request.acceptsLanguages(langs)`

如果没有提供类型，则返回 __所有__ 可接受的类型。

如果提供多种类型，将返回最佳匹配。 如果没有找到匹配项，则返回一个`false`，你应该向客户端发送一个`406 "Not Acceptable"` 响应。

如果接收到任何类型的接收头，则会返回第一个类型。 因此，你提供的类型的顺序很重要。

### request.accepts(types)

检查给定的 `type(s)` 是否可以接受，如果 `true`，返回最佳匹配，否则为 `false`。 `type` 值可能是一个或多个 mime 类型的字符串，如 `application/json`，扩展名称如 `json`，或数组 `["json", "html", "text/plain"]`。

```js
// Accept: text/html
ctx.accepts('html');
// => "html"

// Accept: text/*, application/json
ctx.accepts('html');
// => "html"
ctx.accepts('text/html');
// => "text/html"
ctx.accepts('json', 'text');
// => "json"
ctx.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
ctx.accepts('image/png');
ctx.accepts('png');
// => false

// Accept: text/*;q=.5, application/json
ctx.accepts(['html', 'json']);
ctx.accepts('html', 'json');
// => "json"

// No Accept header
ctx.accepts('html', 'json');
// => "html"
ctx.accepts('json', 'html');
// => "json"
```

你可以根据需要多次调用 `ctx.accepts()`，或使用 switch：

```js
switch (ctx.accepts('json', 'html', 'text')) {
  case 'json': break;
  case 'html': break;
  case 'text': break;
  default: ctx.throw(406, 'json, html, or text only');
}
```

### request.acceptsEncodings(encodings)

检查 `encodings` 是否可以接受，返回最佳匹配为 `true`，否则为 `false`。 请注意，您应该将`identity` 作为编码之一！

```js
// Accept-Encoding: gzip
ctx.acceptsEncodings('gzip', 'deflate', 'identity');
// => "gzip"

ctx.acceptsEncodings(['gzip', 'deflate', 'identity']);
// => "gzip"
```

当没有给出参数时，所有接受的编码将作为数组返回：

```js
// Accept-Encoding: gzip, deflate
ctx.acceptsEncodings();
// => ["gzip", "deflate", "identity"]
```

请注意，如果客户端显式地发送 `identity;q=0`，那么 `identity` 编码（这意味着没有编码）可能是不可接受的。 虽然这是一个边缘的情况，你仍然应该处理这种方法返回 `false` 的情况。

### request.acceptsCharsets(charsets)

检查 `charsets` 是否可以接受，在 `true` 时返回最佳匹配，否则为 `false`。

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets('utf-8', 'utf-7');
// => "utf-8"

ctx.acceptsCharsets(['utf-7', 'utf-8']);
// => "utf-8"
```

当没有参数被赋予所有被接受的字符集将作为数组返回：

```js
// Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
ctx.acceptsCharsets();
// => ["utf-8", "utf-7", "iso-8859-1"]
```

### request.acceptsLanguages(langs)

检查 `langs` 是否可以接受，如果为 `true`，返回最佳匹配，否则为 `false`。

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages('es', 'en');
// => "es"

ctx.acceptsLanguages(['en', 'es']);
// => "es"
```

当没有参数被赋予所有接受的语言将作为数组返回：

```js
// Accept-Language: en;q=0.8, es, pt
ctx.acceptsLanguages();
// => ["es", "pt", "en"]
```

### request.idempotent

检查请求是否是幂等的。

### request.socket

返回请求套接字。

### request.get(field)

返回请求标头。
