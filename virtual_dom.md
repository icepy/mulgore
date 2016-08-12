# Virtual DOM

> 什么是Virtual DOM（虚拟DOM）

从一些公开的资料来看，React提供了高效的虚拟DOM以及diff算法，才让React之所以那么快。在内存中的对象，从某种意义上来说，这是非常快速的。如果你使用 `JSX` 并且阅读过编译之后的代码，我想你应该看过如下的代码：

```JavaScript
React.createElement(
	'div',
	{
	  className: css
	},
	React.createElement(
	   'label',
	   { className: 'item-label' },
	    label
	),
	React.createElement(
	   'div',
	      { className: 'item-field' },
	      Inputs
	)
);
```

是的，`JSX` 只是一个语法糖，看起来我们在书写 HTML，实际上通过工具将其转化成了JS的某一个对象上的某一个方法。我们可以将其转化成更简单的对象，我想你应该就可以看明白了：

```JavaScript
var Element = {
	type: 'div',
	props: {
		className: css
	},
	chidlren:[
		{
			type: 'label',
			props: {
				className: 'item-label'
			},
			children: [
				label
			]
		},
		{
			type: 'div',
			props: {
				className: 'item-field'
			},
			children: [
				Inputs
			]
		}
	]
};

```

而这正是在内存中所描述的 Virtual DOM，当你需要获取某个节点下的数据时，你可以不需要再像以前那样 `document.getElementById('div').innerHTML` 了，而是直接从内存中来获取数据 `this.props` 以及 ref对象。

这就是为什么React那么快速的原因之一。

