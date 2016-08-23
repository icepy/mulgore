# React生命周期的实现

根据上一下节我们可以知道如果你的组件是继承于 React.Component，那么它会进入 ReactCompositeComponent，这是一个专门处理生命周期的逻辑，你可以阅读 `src/renderers/shared/reconciler/ReactCompositeComponent.js` 文件，它的起始调用栈是通过一个事务进入了 `mountComponentIntoNode` 函数，在这个函数中最后进入 `mountComponent` 方法中。

> 中间经过了一个非常简单的方法，感觉起来就是一个中转的 mountComponent，可以在 src/renderers/shared/reconciler/ReactReconciler.js 文件中阅读。

## mountComponent

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

进入了 `performInitialMount` 方法中，在这个方法内它调用了 `componentWillMount` 也就是在组件挂载之前的生命周期钩子方法。然后通过 `renderedElement = this._renderValidatedComponent();` 来获取 ReactElement 创建的对象，并且通过 `this._renderedComponent = this._instantiateReactComponent(
      renderedElement
    );` 来获取组件实例，然后继续调用了 `ReactReconciler.mountComponent` ，其实又回到了 `mountComponent` 中，如此循环下去，一直到最后，将组件都处理完毕。

```JavaScript
 if (inst.componentDidMount) {
   if (__DEV__) {
    transaction.getReactMountReady().enqueue(invokeComponentDidMountWithTimer, this);
   } else {
    transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
   }
 }
```

以及`componentDidMount` 方法是加入到了一个事务中的队列内。通过源码我们可以很清晰的看见一个很有趣的现象，父组件的 `componentWillMount` 方法一定在子组件的 `componentWillMount` 方法之前调用。父组件的 `componentDidMount` 方法一定在子组件的 `componentDidMount`方法之后调用。当然，如果你在 `componentWillMount` 中使用了 `setState` 方法，那么这又会进入另外一个流程，不过你可以放心它肯定不会调用 `render` 方法，而是去合并 state ，这一个步骤要在 update 中描述。

```JavaScript
if (this._pendingStateQueue) {
  inst.state = this._processPendingState(inst.props, inst.context);
}
```

目前为止，React生命周期的前一部分处理完毕。

## updateComponent

update 阶段是对于调用了 `setState` 方法时的处理，这也意味着你需要更新组件。不过这一个阶段，你需要理解的是当你调用 `setState` 时并不是意味着进入更新，而是它将 state 加入了一个临时队列（关于 state 的部分，会在后续的更新机制中说明），不过它的调用栈流程，会进入 `updateComponent` 方法中。

在这个方法中首先处理的是 `componentWillReceiveProps` 方法，然后才会处理 `shouldComponentUpdate` 并且将前后两个 props 传递给你，根据你的返回值来决定是否调用 `_performComponentUpdate` 方法，如果你返回的为true，那么你的组件会进入 `componentWillUpdate`，最后开始执行 `_updateRenderedComponent` 来重新渲染组件。

最后当你的组件渲染完成之后，则会进入 `componentDidUpdate` ，当然前提是这个方法是存在的。`_pendingForceUpdate` 是非常重要的一个状态，你只需要知道，当这个状态为false时并且 `shouldComponentUpdate`存在时它才会调用。

```JavaScript

//更新组件，__DEV__ 可忽略。

var inst = this._instance;
var willReceive = false;
var nextContext;
var nextProps;

// Determine if the context has changed or not
if (this._context === nextUnmaskedContext) {
  nextContext = inst.context;
} else {
  nextContext = this._processContext(nextUnmaskedContext);
  willReceive = true;
}

// Distinguish between a props update versus a simple state update
if (prevParentElement === nextParentElement) {
  // Skip checking prop types again -- we don't read inst.props to avoid
  // warning for DOM component props in this upgrade
  nextProps = nextParentElement.props;
} else {
  nextProps = this._processProps(nextParentElement.props);
  willReceive = true;
}

// An update here will schedule an update but immediately set
// _pendingStateQueue which will ensure that any state updates gets
// immediately reconciled instead of waiting for the next batch.
if (willReceive && inst.componentWillReceiveProps) {
  if (__DEV__) {
    if (this._debugID !== 0) {
      ReactInstrumentation.debugTool.onBeginLifeCycleTimer(
        this._debugID,
        'componentWillReceiveProps'
      );
    }
  }
  inst.componentWillReceiveProps(nextProps, nextContext);
  if (__DEV__) {
    if (this._debugID !== 0) {
      ReactInstrumentation.debugTool.onEndLifeCycleTimer(
        this._debugID,
        'componentWillReceiveProps'
      );
    }
  }
}

var nextState = this._processPendingState(nextProps, nextContext);
var shouldUpdate = true;

if (!this._pendingForceUpdate && inst.shouldComponentUpdate) {
  if (__DEV__) {
    if (this._debugID !== 0) {
      ReactInstrumentation.debugTool.onBeginLifeCycleTimer(
        this._debugID,
        'shouldComponentUpdate'
      );
    }
  }
  shouldUpdate = inst.shouldComponentUpdate(nextProps, nextState, nextContext);
  if (__DEV__) {
    if (this._debugID !== 0) {
      ReactInstrumentation.debugTool.onEndLifeCycleTimer(
        this._debugID,
        'shouldComponentUpdate'
      );
    }
  }
}

if (__DEV__) {
  warning(
    shouldUpdate !== undefined,
    '%s.shouldComponentUpdate(): Returned undefined instead of a ' +
    'boolean value. Make sure to return true or false.',
    this.getName() || 'ReactCompositeComponent'
  );
}

this._updateBatchNumber = null;
if (shouldUpdate) {
  this._pendingForceUpdate = false;
  // Will set `this.props`, `this.state` and `this.context`.
  this._performComponentUpdate(
    nextParentElement,
    nextProps,
    nextState,
    nextContext,
    transaction,
    nextUnmaskedContext
  );
} else {
  // If it's determined that a component should not update, we still want
  // to set props and state but we shortcut the rest of the update.
  this._currentElement = nextParentElement;
  this._context = nextUnmaskedContext;
  inst.props = nextProps;
  inst.state = nextState;
  inst.context = nextContext;
}

.....
// _performComponentUpdate 调用栈中

this._updateRenderedComponent(transaction, unmaskedContext);

//如果存在 componentDidUpdate
transaction.getReactMountReady().enqueue(
   inst.componentDidUpdate.bind(inst, prevProps, prevState, prevContext),
   inst
);

```

当你进入 `_updateRenderedComponent` 流程时，于是又开始了 `updateComponent` 的调用，这里跟 `mountComponent` 的流程有些类似。父组件的 `componentWillUpdate` 一定会在子组件的 `componentWillUpdate` 之前调用，父组件的 `componentDidUpdate` 一定会在子组件的 `componentDidUpdate` 之后调用。

## unmountComponent

`unmountComponent`的调用必然是在你调用了 `setState` 方法之后并且需要更新组件时，因为这了牵扯到了 `setState` 更新机制，详细的会在后期慢慢描述，在这个生命周期中，你只需要注意一个方法 `componentWillUnmount`。

你可以在源码中看见，很多变量都设置成了null。

