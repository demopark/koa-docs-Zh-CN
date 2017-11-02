# 常见问题

## Koa 替代 Express?

它更像是 Connect，但是很多 Express 的好东西被转移到 Koa 的中间件级别，以帮助形成更强大的基础。 这使得中间件对于整个堆栈而言不仅仅是最终应用程序代码，而且更易于书写，并更不容易出错。

通常，许多中间件将重新实现类似的功能，甚至更糟的是不正确地实现它们， 如签名的cookie 加密等通常是应用程序特定的，而不是中间件特定的。

## Koa 替代 Connect?

不，只是不同的功能，现在通过构建器也可以让我们用较少的回调编写代码。 Connect 同样可以，有些人可能仍然喜欢它，这取决于你喜欢什么。

## Koa 包含路由吗?

不 - Koa 没有开箱即用的路由, 但是有很多路由中间件可用: https://github.com/koajs/koa/wiki

## 为什么 Koa 不是 Express 4.0?

Koa 与现在所知的 Express 差距很大，设计根本上有很大差异，所以从 Express 3.0 迁移到Express 4.0 将有意味着重写整个应用程序，所以我们考虑创建一个新的库。

## Koa 对象有什么自定义属性？

Koa 使用它的自定义对象: `ctx`, `ctx.request`, 和 `ctx.response`.
这些对象使用便捷的方法和 getter/setter 来抽象 node 的 `req` 和 `res` 对象。

通常，添加到这些对象的属性必须遵循以下规则：

  - 它们必须是非常常用的 和/或 必须做一些有用的事情
  - 如果一个属性作为一个 setter 存在，那么它也将作为一个 getter 存在，但反之亦然

许多 `ctx.request` 和 `ctx.response` 的属性都被委托给 `ctx`。
如果它是一个 getter/setter，那么 getter 和 setter 都将严格对应于 `ctx.request` 或 `ctx.response`。

附加其他属性之前，请考虑这些规则。
