# 从 Koa v1.x 迁移到 v2.x

## 新的中间件签名

Koa v2 引入了新的中间件签名。

**旧签名中间件（v1.x）支持将在 v3 中删除**

新的中间件签名是这样的:

```js
// 使用异步箭头方法
app.use(async (ctx, next) => {
  try {
    await next() // next 现在是一个方法
  } catch (err) {
    ctx.body = { message: err.message }
    ctx.status = err.status || 500
  }
})

app.use(async ctx => {
  const user = await User.getById(this.session.userid) // await 替换了 yield
  ctx.body = user // ctx 替换了 this
})
```

你不必一定使用异步函数 - 你只需要传递一个返回 promise 的函数。返回 promise 的常规方法也可以使用！

签名已更改为通过 `ctx` 取代 `this` 显式参数传递 `Context`。

上下文传递更改使得 koa 更能兼容 es6 的箭头函数，通过捕获 “this”。


## 在 v2.x 中使用 v1.x 中间件

Koa v2.x将尝试转换 `app.use` 上的旧签名，生成器中间件， 使用 [koa-convert](https://github.com/koajs/convert).
不过建议您选择尽快迁移所有 v1.x 中间件。

```js
// Koa 将转换
app.use(function *(next) {
  const start = Date.now();
  yield next;
  const ms = Date.now() - start;
  console.log(`${this.method} ${this.url} - ${ms}ms`);
});
```

您也可以手动执行，在这种情况下，Koa不会转换。

```js
const convert = require('koa-convert');

app.use(convert(function *(next) {
  const start = Date.now();
  yield next;
  const ms = Date.now() - start;
  console.log(`${this.method} ${this.url} - ${ms}ms`);
}));
```

## 升级中间件

您将不得不使用新的中间件签名将您的生成器转换为异步功能：

```js
app.use(async (ctx, next) => {
  const user = await Users.getById(this.session.user_id);
  await next();
  ctx.body = { message: 'some message' };
})
```

升级中间件可能需要一些工作。 一个迁移方式是逐个更新它们。

1. 将所有当前的中间件包装在 `koa-convert` 中
2. 测试
3. `npm outdated` 看看哪个 koa 中间件已经过时了
4. 更新一个过时的中间件，使用 `koa-convert` 删除
5. 测试
6. 重复步骤3-5，直到完成

## 升级你的代码

您应该开始重构代码，以便轻松迁移到 Koa v2：

- 各处都是 promises 返回!
- 不要使用 `yield*`
- 不要使用 `yield {}` 或 `yield []`.
  - 转换 `yield []` 为 `yield Promise.all([])`
  - 转换 `yield {}` 为 `yield Bluebird.props({})`

您也可以重构 Koa 中间件功能之外的逻辑。 创建一个方法像 `function* someLogic(ctx) {}` 然后在你的中间件中调用 `const result = yield someLogic(this)`.
不使用 `this` 将有助于迁移到新的中间件签名，所以不使用 `this`。

## 应用对象构造函数需要 new

在 v1.x 中，可以直接调用应用构造函数，而不用 `new` 实例化一个应用程序的实例。 例如：

```js
var koa = require('koa');
var app = module.exports = koa();
```

v2.x 使用 es6 类，需要使用 `new` 关键字。

```js
var koa = require('koa');
var app = module.exports = new koa();
```

## 删除 ENV 特定的日志记录行为

对于 `test` 环境的显式检查从错误处理中删除。

## 依赖变化

- [co](https://github.com/tj/co) 不再与Koa捆绑在一起。直接 require  或 import 它.
- [composition](https://github.com/thenables/composition) 不再使用并已废弃。

## v1.x 支持

仍然支持 v1.x 分支，但应该不会得到功能性更新。 除了此迁移指南外，文档将针对最新版本。

## 帮帮忙

如果您遇到本迁移指南未涉及的迁移相关问题，请考虑提交文档提取请求。
