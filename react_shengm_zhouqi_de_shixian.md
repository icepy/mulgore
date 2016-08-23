# React生命周期的实现

根据上一下节我们可以知道如果你的组件是继承于 React.Component，那么它会进入 ReactCompositeComponent，这是一个专门处理生命周期的逻辑，你可以阅读 `src/renderers/shared/reconciler/ReactCompositeComponent.js` 文件，它的起始调用栈是通过一个事务进入了 `mountComponentIntoNode` 函数，在这个函数中最后进入 `mountComponent` 方法中。

> 中间经过了一个非常简单的方法，感觉起来就是一个中转的 mountComponent，可以在 src/renderers/shared/reconciler/ReactReconciler.js 文件中阅读。

在 `mountComponent` 中 很大一段都是处理警告信息以及初始化的动作，重点在于：

```JavaScript
if (inst.unstable_handleError) {
  markup = this.performInitialMountWithErrorHandling(
    renderedElement,
    nativeParent,
    nativeContainerInfo,
    transaction,
    context
  );
} else {
  markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction, context);
}
```

进入了 `performInitialMount` 方法中，在这个方法内它调用了 `componentWillMount` 也就是在组件挂载之前的生命周期钩子方法。然后通过 `renderedElement = this._renderValidatedComponent();` 来获取了ReactElement 创建了对象。	然后继续调用了 `ReactReconciler.mountComponent` ，其实又回到了 `mountComponent` 中，如此循环下去，一直到最后，将组件都处理完毕。

```JavaScript
 if (inst.componentDidMount) {
   if (__DEV__) {
    transaction.getReactMountReady().enqueue(invokeComponentDidMountWithTimer, this);
   } else {
    transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
   }
 }
```
以及`componentDidMount` 方法是加入到了一个事务中的队列内。

目前为止，React生命周期的前一部分处理完毕。