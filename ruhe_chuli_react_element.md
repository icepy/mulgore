# 如何处理 React 元素对象

> 在上一小节中出现的 React.CreactElement，它是什么？

从第一章中你已经学会了如何查询文件，之后将直接贴出路径。`CreactElement`, `ReactElement` 在 `src/isomorphic/classic/element/ReactElement.js`。

## CreateElement

```JavaScript
ReactElement.createElement = function(type, config, children) {

}
```

从这个静态方法来看其实已经很像之前那个普通对象的描述了，它也接收三个参数，分别是`type`，`config`，`children`。在这个方法的顶部，分别定义了一些变量，不过这些都不重要，你可以继续往下阅读，第一个 `if` 判断当 config 参数不等于 null时候执行，在这一部分中分别处理了一些比如 `ref`，`props` 等。

接着剔除前两个参数来获取所有的子节点对象 `var childrenLength = arguments.length - 2;` ，并且将其装载到 `props.children`中，有意思的是它根据childrenLength的长度来生成一个空数组，然后将子节点添加到这个数组中，最后赋值给 `props.children`。最后开始处理 `defaultProps` 并装载到 `props` 对象中，最后return 出来一个 `ReactElement`。

*注明：__DEV__中的一些处理主要是辅助开发阶段，可以得到一些有用的信息。warning 在fbjs中，有兴趣的可以到node_modules中查找。*


## ReactElement

在 `CreactElement` 中我们可以知道它 return 了一个 `ReactElement`，那它又是什么呢？

```JavaScript
var ReactElement = function(type, key, ref, self, source, owner, props) {

}
```

第一眼看去 `ReactElement` 比 `CreactElement` 多了很多参数，不过不要紧，从初始化的角度来看，其实很多参数都是 `null` ，比如 `owner` 描述的是当前节点，初始化时从 `ReactCurrentOwner` 读取，阅读 `src/isomorphic/classic/element/ReactCurrentOwner.js` 其实它只定义了一个current 等于 null 的模块。

```JavaScript
var ReactCurrentOwner = {

  /**
   * @internal
   * @type {ReactComponent}
   */
  current: null,

};

module.exports = ReactCurrentOwner;
``` 

`__DEV__` 部分如果你不想阅读的话，整个 `ReactElement` 只是存在了一个 `element` 对象，并将其 return 了出来。

```JavaScript
var element = {
  // This tag allow us to uniquely identify this as a React Element
  $$typeof: REACT_ELEMENT_TYPE,
  // Built-in properties that belong on the element
  type: type,
  key: key,
  ref: ref,
  props: props,
  // Record the component responsible for creating this element.
  _owner: owner,
};
```

`REACT_ELEMENT_TYPE` 这个常量使用 `Symbol` 来判断是否为 `React` 元素类型。

## 如何装载

从 `CreactElement` 和 `ReactElement` 来看其实这是一个非常简单的对象组合的过程，最后你应该可以得到如下的一个对象：

```JavaScript
{
	$$typeof: (typeof Symbol === 'function' && Symbol.for && Symbol.for('react.element')) ||
  0xeac7,
	type: 'div',
	key: null,
	ref: null,
	props: {
		className: 'css',
		children: [
			{
				$$typeof: (typeof Symbol === 'function' && Symbol.for && Symbol.for('react.element')) ||
  0xeac7,
  				type: 'label',
  				key: null,
  				ref: null,
  				props: {
  					className: 'item-label',
  					children: [] || object // 如果你的子节点只有一个，children将不适数组而是一个对象
  				},
  				_owner: null
			}
		]
	},
	_owner: null
}
```

有了它我们还并不能直接在浏览器中访问，因为你还需要一个 `render` 的过程，这个 `render` 方法存在着两个入口，我比较喜欢将其称为 `初始化入口` 和 `元素入口`。

> 初始化入口 src/renderers/dom/client/ReactMount.js

其实 `元素入口` 什么事情都没有做，只不过是帮助你调用了一下 render 方法，然后 return 出来 `React.CreactElement`。

当你在初始化入口写render时，实际上是进入了如下的一个方法中：

```JavaScript
render: function(nextElement, container, callback) {

}
```

根据其调用栈你可以看见这个 render 方法又调用了 _renderSubtreeIntoContainer 方法，看起来这才是重头戏。

```JavaScript
_renderSubtreeIntoContainer: function(parentComponent, nextElement, container, callback) {

}
```

在经过一系列的处理，包括 container 对象，最后调用 `_renderNewRootComponent` 来生存一个 component 并返回。

