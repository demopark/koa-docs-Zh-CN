# Koa 与 Express

在理念上，Koa 旨在 “修复和替换节点”，而 Express 旨在 “增加节点”。
Koa 使用承诺和异步功能来摆脱回调地狱的应用程序，并简化错误处理。
它暴露了自己的 `ctx.request` 和 `ctx.response` 对象，而不是 node 的 `req` 和 `res` 对象。

另一方面，Express 通过附加的属性和方法增加了 node 的 `req` 和 `res` 对象，并且包含许多其他 “框架” 功能，如路由和模板，而 Koa 则没有。

因此，Koa 可被视为 node.js 的 `http` 模块的抽象，其中 Express 是 node.js 的应用程序框架。

| 功能           | Koa | Express | Connect |
|------------------:|-----|---------|---------|
| Middleware Kernel | ✓   | ✓       | ✓       |
| Routing           |     | ✓       |         |
| Templating        |     | ✓       |         |
| Sending Files     |     | ✓       |         |
| JSONP             |     | ✓       |         |

因此，如果您想要更接近 node.js 和传统的 node.js 样式编码，那么您可能希望坚持使用Connect/Express 或类似的框架。
如果你想摆脱回调，请使用 Koa。

由于这种不同的理念，其结果是传统的 node.js “中间件”（即“（req，res，next）”的函数）与Koa不兼容。 你的应用基本上要重新改写了。

## Koa 替代 Express?

它更像是 Connect，但是很多 Express 的好东西被转移到 Koa 的中间件级别，以帮助形成更强大的基础。 这使得中间件对于整个堆栈而言不仅仅是最终应用程序代码，而且更易于书写，并更不容易出错。

通常，许多中间件将重新实现类似的功能，甚至更糟的是不正确地实现它们， 如签名的cookie 加密等通常是应用程序特定的，而不是中间件特定的。

## Koa 替代 Connect?

不，只是不同的功能，现在通过构建器也可以让我们用较少的回调编写代码。 Connect 同样可以，有些人可能仍然喜欢它，这取决于你喜欢什么。

## 为什么 Koa 不是 Express 4.0?

Koa 与现在所知的 Express 差距很大，设计根本上有很大差异，所以从 Express 3.0 迁移到Express 4.0 将有意味着重写整个应用程序，所以我们考虑创建一个新的库。

## Koa 与 Connect/Express 有哪些不同?

### 基于 Promises 的控制流程

没有回调地狱。

通过 try/catch 更好的处理错误。

无需域。

### Koa 非常精简

不同于 Connect 和 Express, Koa 不含任何中间件.

不同于 Express, 不提供路由.

不同于 Express, 不提供许多便捷设施。 例如，发送文件.

Koa 更加模块化.

### Koa 对中间件的依赖较少

例如, 不使用 “body parsing” 中间件，而是使用 body 解析函数。

### Koa 抽象 node 的 request/response

减少攻击。

更好的用户体验。

恰当的流处理。
  
### Koa 路由（第三方库支持）

由于 Express 带有自己的路由，而 Koa 没有任何内置路由，但是有 koa-router 和 koa-route 第三方库可用。同样的, 就像我们在 Express 中有 helmet 保证安全, 对于 koa 我们有 koa-helmet 和一些列的第三方库可用。
