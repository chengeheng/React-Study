# React-Fiber

## 特征

能够将渲染工作分割成块，并将其分散到多个帧中。

## 影响

- `componentWillMount`、`componentWillReceiveProps`、`componentWillUpdate`这几个生命周期不再安全，可能会被多次执行
- 需要关注下 react 为任务片设置的优先级，特别是页面用动画的情况

## 目标

Fiber 的主要目标是使 React 能够利用 scheduling，具体来说就是：

- 暂停工作，稍后再回来
- 为不同类型的工作分配优先权
- 重复以前完成的工作
- 如果不再需要，请终止工作

为了做到这一点，首先需要一种把工作分解成单元的方法，从某种意义上说，这就是`fiber`，`fiber`代表一个工作单元。

## fiber 源码中的一些数据结构

- `fiber`是个链表，有`child`和`sibing`属性，指向第一个子节点和相邻的兄弟节点，`return`指向器父节点，从而构成`fiber tree`。
- `updateQueue`（更新队列）也是一个链表，有个`first`和`last`两个属性，指向第一个和最后一个`update`对象。
- 每个`fiber`有一个属性`updateQueue`指向其对应的更新队列。
- 每个`fiber`有一个属性`alternate`，开始时指向一个自己的 clone 体，`update`的变化会先更新到`alternate`上，当更新结束，`alternate`会替换当前的`current`。
