# React （顶层API）
React 是 React 库的入口。
如果你通过使用 script 标签的方式来加载 React，则可以通过 React 全局变量对象来获得 React 的顶层 API。
当你使用 ES6 与 npm 时，可以通过编写 import React from 'react' 来引入它们。
当你使用 ES5 与 npm 时，则可以通过编写 var React = require('react') 来引入它们。

## 组件 相关 
### React.Component
### React.PureComponent
### React.memo
## 创建元素 相关 
我们建议使用 JSX 来编写你的 UI 组件。
每个 JSX 元素都是调用 React.createElement() 的语法糖。
一般来说，如果你使用了 JSX，就不再需要调用以下方法。

## createElement()
```
React.createElement(
  type,
  [props],
  [...children]
)
```
创建并返回指定类型的新 React 元素。其中的类型参数既可以是标签名字符串（如 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

使用 JSX 编写的代码将会被转换成使用 React.createElement() 的形式。如果使用了 JSX 方式，那么一般来说就不需要直接调用 React.createElement()。请查阅不使用 JSX 章节获得更多信息。

## createFactory()
## 转换元素 相关 
## cloneElement()
## isValidElement()
## React.Children
## Fragments 相关 
React 还提供了用于减少不必要嵌套的组件。
## React.Fragment
## Refs 相关 
## React.createRef
## React.forwardRef
## Suspense 相关 
Suspense 使得组件可以“等待”某些操作结束后，再进行渲染。目前，Suspense 仅支持的使用场景是：通过 React.lazy 动态加载组件。它将在未来支持其它使用场景，如数据获取等。
## React.lazy
## React.Suspense

# React.Component
react-dom 的 package 提供了可在应用顶层使用的 DOM（DOM-specific）方法，如果有需要，你可以把这些方法用于 React 模型以外的地方。不过一般情况下，大部分组件都不需要使用这个模块。

## render()
在提供的 container 里渲染一个 React 元素，并返回对该组件的引用（或者针对无状态组件返回 null）。

```
ReactDOM.render(element, container[, callback])
```
- element：React 元素
- container：容器节点
- callback：可选的回调函数，该回调将在组件被渲染或更新之后被执行

```
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

## hydrate()

## unmountComponentAtNode()
从 DOM 中卸载组件，会将其事件处理器（event handlers）和 state 一并清除。如果指定容器上没有对应已挂载的组件，这个函数什么也不会做。如果组件被移除将会返回 true，如果没有组件可被移除将会返回 false。
```
ReactDOM.unmountComponentAtNode(container)
```

## findDOMNode()
## createPortal()
