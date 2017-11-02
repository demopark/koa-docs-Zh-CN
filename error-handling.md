# 错误处理

## Try-Catch

使用 async 方法意味着你可以 try-catch `next`.
此示例为所有错误添加了一个 `.status`：

  ```js
  app.use(async (ctx, next) => {
    try {
      await next();
    } catch (err) {
      err.status = err.statusCode || err.status || 500;
      throw err;
    }
  });
  ```

### 默认错误处理程序

默认的错误处理程序本质上是中间件链开始时的一个 try-catch。要使用不同的错误处理程序，只需在中间件链的起始处放置另一个 try-catch，并在那里处理错误。但是，默认错误处理程序对于大多数用例来说都是足够好的。它将使用状态代码 `err.status`，或默认为500。如果 `err.expose` 是 true，那么 `err.message` 就是答复。否则，将使用从错误代码生成的消息（例如，对于代码500，将使用消息“内部服务器错误”）。所有标头将从请求中清除，但是任何在 `err.headers` 中的标头将会被设置。你可以使用如上所述的 try-catch 来向此列表添加标头。

以下是创建你自己的错误处理程序的示例：

```js
app.use(async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    // will only respond with JSON
    ctx.status = err.statusCode || err.status || 500;
    ctx.body = {
      message: err.message
    };
  }
})
```

## 错误事件

错误事件侦听器可以用 `app.on('error')` 指定。如果未指定错误侦听器，则使用默认错误侦听器。错误侦听器接收所有中间件链返回的错误，如果一个错误被捕获并且不再抛出，它将不会被传递给错误侦听器。如果没有指定错误事件侦听器，那么将使用 `app.onerror`，如果 `error.expose` 为 true 并且 `app.silent` 为 false，则简单记录错误。
