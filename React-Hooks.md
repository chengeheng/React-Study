# React-Hooks

> why - what - how

## 设计动机

要理解`Hooks`的设计动机，就要对类组件和函数组件两种组件形式进行思考。

**类组件 -> 函数组件**

### 类组件与函数组件对比

- 类组件需要继承 class，函数组件不需要
- 类组件可以访问生命周期方法，函数组件不能
- 类组件可以获取实例化后的 this，并基于这个 this 做各种各样的事情，而函数组件不可以
- 类组件可以定义并维护 state（状态），函数组件不可以（所以函数组件又叫无状态组件）

类组件是面向对象编程思想的一种表征，封装和继承。**功能全而强大，但是过于繁重，并且内部逻辑复用困难。**

函数组件肉眼可见的特质自然包括轻量、灵活、易于组织和维护、较低的学习成本等。**函数组件会捕获 render 内部的状态，这是两类组件最大的不同。**

类组件和函数组件的差别更类似与面向对象和函数式编程这两套不同的设计思想之间的差异。

**React 组件本身的定位就是函数，一个吃进数据、吐出 UI 的函数。**

**React 框架的主要工作，就是及时地把声明式的代码转换为命令式的 DOM 操作，把数据层面的描述映射到用户可见的 UI 变化中去。**

React 的数据应该总是紧紧地和渲染绑定在一起的，而类组件做不到这一点。类组件中的 this 是获取的是最新状态，当有进行延时操作时，真正触发的 this 对象已经不是原来的 this 了。

而函数组件真正的把数据和渲染绑定到一起，在函数触发的时候创建的上下文就是当前的上下文，不会因为延时而获取最新的上下文。

## Hooks 的本质：一套能够使函数组件更强大、更灵活的钩子

对比函数组件和类组件之后，可以体会到函数组件的优势。而 Hooks 就是帮助函数式组件补齐对比类组件“少”的那些东西，比如生命周期、对 state 的管理等。

同时 Hooks 允许自由的选择和使用需要的能力，方便的代码逻辑的封装和复用。

## 为什么要使用 React-hooks

- 告别难以理解的 Class
  - 主要是 `this` 和`生命周期`
  - this 需要手动绑定到实例上（bind、箭头函数）
  - 生命周期，学习成本高
- 解决业务逻辑难以拆分的问题

  不合理的逻辑规划方式；逻辑一度与生命周期耦合在一起，同一个生命周期会处理多个逻辑，难以理解

- 使状态逻辑复用变得简单可行

  原来靠 HOC（高阶组件）和 Render Props 这些组件设计模式，现在可以通过自定义 Hook，达到既不破坏组件结构、又能实现逻辑复用的效果。

- 函数组件从设计思想上看，更加契合 React 的理念 -- 参考类组件和函数组件的对比

## Hooks 的局限性

- Hooks 暂时还不能完全为函数组件补齐类组件的能力（少部分生命周期）
- “轻量”几乎是函数组件的基因，这可能会使它不能够很好地消化“复杂”

  函数组件给了我们一定程度的自由，却也对开发者的水平提出了更高的要求。

  > 耦合和内聚的边界

- Hooks 在使用层面有着严格的规则约束

## 官方定义的使用原则

- 只在 React 函数中调用 Hook
- 不要在循环、条件或嵌套函数中调用 Hook：**确保 Hooks 在每次渲染时都保持同样的执行顺序**

```javascript
// 借用修言课程中的代码
import React, { useState } from "react";
// isMounted 用于记录是否已挂载（是否是首次渲染）
let isMounted = false;
function PersonalInfoComponent() {
	// 定义变量的逻辑不变
	let name, age, career, setName, setCareer;
	// 这里追加对 isMounted 的输出，这是一个 debug 性质的操作
	console.log("isMounted is", isMounted);
	// 这里追加 if 逻辑：只有在首次渲染（组件还未挂载）时，才获取 name、age 两个状态
	if (!isMounted) {
		// eslint-disable-next-line
		[name, setName] = useState("修言");
		// eslint-disable-next-line
		[age] = useState("99");
		// if 内部的逻辑执行一次后，就将 isMounted 置为 true（说明已挂载，后续都不再是首次渲染了）
		isMounted = true;
	}
	// 对职业信息的获取逻辑不变
	[career, setCareer] = useState("我是一个前端，爱吃小熊饼干");
	// 这里追加对 career 的输出，这也是一个 debug 性质的操作
	console.log("career", career);
	// UI 逻辑的改动在于，name和age成了可选的展示项，若值为空，则不展示
	return (
		<div className="personalInfo">
			{name ? <p>姓名：{name}</p> : null}
			{age ? <p>年龄：{age}</p> : null}
			<p>职业：{career}</p>
			<button
				onClick={() => {
					setName("秀妍");
				}}
			>
				修改姓名
			</button>
		</div>
	);
}
export default PersonalInfoComponent;

// 报错，Rendered fewer hooks than expected。组件渲染的hooks比期望的更少。
```

> eslint-disable-next-line:禁用下一行的 eslint 校验

**从源码调用流程看原理：Hooks 的正常运作，在底层依赖于顺序链表**

hook 相关的所有信息收敛在一个 hook 对象里，而 hook 对象之间以单向链表的形式相互串联。hooks 的渲染是通过“依次遍历”来定位每个 hooks 内容的。如果前后两次读到的链表在顺序上出现差异，那么渲染的结果自然是不可控的。

**Hooks 的本质其实就是链表**
