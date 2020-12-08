# RequestIdleCallback

## 页面流畅与 FPS

在浏览器中，页面是一帧一帧绘制出来的，一般的浏览器每秒钟会绘制 60 帧（FPS），也就是每帧需要 16ms 左右，人眼在这个时间内是无法感受到绘制的过程的，这时页面就是流畅的。如果 js 长时间占用主进程，会导致一些 ui 无法及时得到渲染，页面就会出现卡顿的效果。

通常一帧内需要完成以下六个步骤的任务：

- 处理用户的交互
- JS 解析执行
- 帧开始。窗口尺寸变更，页面滚动等处理
- requestAnimationFrame（rAF）
- 布局
- 绘制

**上面步骤如果在 16ms 内完成，说明一帧的时间内还有富余，此时就会执行 requestIdleCallback 里面注册的任务**

## 语法

```javascript
const handle = window.requestIdleCallback(callback[, options]);
```

- callback: 回调，表示空闲时间需要执行的任务。该回调函数接受一个`IdleDeadline`对象作为入参，这个对象包含：

  - didTimeout，布尔值，表示任务是否超时，结合 timeRemaining 使用。
  - timeReamaining， 方法，表示当前帧剩余的时间，也可理解为留给任务的时间还有多少。

- options：目前 options 只有一个参数

  - timeout：表示超过这个时间后，如果任务还没执行，则强制执行，不必等待空闲时间

> 参考 demo/RequestIdleCallback.html

## cancelIdleCallback

与`clearTimeout`类似，`requestIdleCallback`会返回一个唯一的 id，可通过`cancelIdleCallback`来取消任务

## 总结

- 一些低优先级的任务可以使用`requestIdleCallback`来等浏览器不忙的时候来执行
- 因为执行的时间有限，`requestIdleCallback`执行的任务应该是尽量能够量化细分的微任务（micro task），执行时间过长的任务会延长帧的时间，导致卡顿
- 不建议在`requestIdleCallback`中操作 DOM

  因为它发生在一帧的最后，此时页面布局已经完成，操作 DOM 会导致页面再次重绘，**DOM 操作建议在`rAF`中执行**

- promise 也不建议在这里面执行，

  因为 Promise 的回调属性在 Event Loop 中是一种优先级较高的微任务，会在`requestIdleCallback`结束后立即执行，不管此时是否还有富余的时间，这样有很大可能会让一帧超过 16ms
