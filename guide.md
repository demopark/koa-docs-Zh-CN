
# 指南

本指南涵盖的 Koa 主题不与 API 直接相关，例如编写中间件的最佳做法和应用程序结构建议。在这些例子中，我们使用 async 函数作为中间件 - 您也可以使用 commonFunction 或generatorFunction，这将些所不同。

## 编写中间件

Koa 中间件是简单的函数，它返回一个带有签名 (ctx, next) 的`MiddlewareFunction`。当中间件运行时，它必须手动调用 `next()` 来运行 “下游” 中间件。

例如，如果你想要跟踪通过添加 `X-Response-Time` 头字段通过 Koa 传播请求需要多长时间，则中间件将如下所示：

```js
async function responseTime(ctx, next) {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  ctx.set('X-Response-Time', `${ms}ms`);
}

app.use(responseTime);
```

如果您是前端开发人员，您可以将 `next();` 之前的任意代码视为“捕获”阶段，这个简易的 gif 说明了 async 函数如何使我们能够恰当地利用堆栈流来实现请求和响应流：

![koa middleware](middleware.gif)

   1. 创建一个跟踪响应时间的日期
   2. 等待下一个中间件的控制
   3. 创建另一个日期跟踪持续时间
   4. 等待下一个中间件的控制
   5. 将响应主体设置为“Hello World”
   6. 计算持续时间
   7. 输出日志行
   8. 计算响应时间
   9. 设置 `X-Response-Time` 头字段
   10. 交给 Koa 处理响应

接下来，我们将介绍创建 Koa 中间件的最佳做法。

## 中间件最佳实践

本节介绍中间件创作最佳实践，例如中间件接受参数，命名中间件进行调试等等。

### 中间件参数

当创建公共中间件时，将中间件包装在接受参数的函数中，遵循这个约定是有用的，允许用户扩展功能。即使您的中间件 _不_ 接受任何参数，这仍然是保持统一的好方法。

这里我们设计的 `logger` 中间件接受一个 `format` 自定义字符串，并返回中间件本身：

```js
function logger(format) {
  format = format || ':method ":url"';

  return async function (ctx, next) {
    const str = format
      .replace(':method', ctx.method)
      .replace(':url', ctx.url);

    console.log(str);

    await next();
  };
}

app.use(logger());
app.use(logger(':method :url'));
```

### 命名中间件

  命名中间件是可选的，但是在调试中分配名称很有用。

```js
function logger(format) {
  return async function logger(ctx, next) {

  };
}
```

### 将多个中间件与 koa-compose 相结合

有时您想要将多个中间件 “组合” 成一个单一的中间件，便于重用或导出。你可以使用 [koa-compose](https://github.com/koajs/compose)

```js
const compose = require('koa-compose');

async function random(ctx, next) {
  if ('/random' == ctx.path) {
    ctx.body = Math.floor(Math.random() * 10);
  } else {
    await next();
  }
};

async function backwards(ctx, next) {
  if ('/backwards' == ctx.path) {
    ctx.body = 'sdrawkcab';
  } else {
    await next();
  }
}

async function pi(ctx, next) {
  if ('/pi' == ctx.path) {
    ctx.body = String(Math.PI);
  } else {
    await next();
  }
}

const all = compose([random, backwards, pi]);

app.use(all);
```

  

### 响应中间件

中间件决定响应请求，并希望绕过下游中间件可以简单地省略 `next()`。通常这将在路由中间件中，但这也可以任意执行。例如，以下内容将以 “two” 进行响应，但是所有三个都将被执行，从而使下游的 “three” 中间件有机会操纵响应。

```js
app.use(async function (ctx, next) {
  console.log('>> one');
  await next();
  console.log('<< one');
});

app.use(async function (ctx, next) {
  console.log('>> two');
  ctx.body = 'two';
  await next();
  console.log('<< two');
});

app.use(async function (ctx, next) {
  console.log('>> three');
  await next();
  console.log('<< three');
});
```

以下配置在第二个中间件中省略了`next()`，并且仍然会以 “two” 进行响应，然而，第三个（以及任何其他下游中间件）将被忽略：

```js
app.use(async function (ctx, next) {
  console.log('>> one');
  await next();
  console.log('<< one');
});

app.use(async function (ctx, next) {
  console.log('>> two');
  ctx.body = 'two';
  console.log('<< two');
});

app.use(async function (ctx, next) {
  console.log('>> three');
  await next();
  console.log('<< three');
});
```

当最远的下游中间件执行 `next();` 时，它实际上是一个 noop 函数，允许中间件在堆栈中的任意位置正确组合。

## 异步操作

Async 方法和 promise 来自 Koa 的底层，可以让你编写非阻塞序列代码。例如，这个中间件从 `./docs` 读取文件名，然后在将给 body 指定合并结果之前并行读取每个 markdown 文件的内容。


```js
const fs = require('fs-promise');

app.use(async function (ctx, next) {
  const paths = await fs.readdir('docs');
  const files = await Promise.all(paths.map(path => fs.readFile(`docs/${path}`, 'utf8')));

  ctx.type = 'markdown';
  ctx.body = files.join('');
});
```

## 调试 Koa

Koa 以及许多构建库，支持来自 [debug](https://github.com/visionmedia/debug) 的 __DEBUG__ 环境变量，它提供简单的条件记录。

例如，要查看所有 koa 特定的调试信息，只需通过 `DEBUG=koa*`，并且在启动时，您将看到所使用的中间件的列表。

```
$ DEBUG=koa* node --harmony examples/simple
  koa:application use responseTime +0ms
  koa:application use logger +4ms
  koa:application use contentLength +0ms
  koa:application use notfound +0ms
  koa:application use response +0ms
  koa:application listen +0ms
```

由于 JavaScript 在运行时没有定义函数名，你也可以将中间件的名称设置为 `._name`。当你无法控制中间件的名称时，这很有用。例如：

```js
const path = require('path');
const serve = require('koa-static');

const publicFiles = serve(path.join(__dirname, 'public'));
publicFiles._name = 'static /public';

app.use(publicFiles);
```

现在，在调试时不只会看到  “serve”，你也会看到：
  Now, instead of just seeing "serve" when debugging, you will see:

```
  koa:application use static /public +0ms
```
