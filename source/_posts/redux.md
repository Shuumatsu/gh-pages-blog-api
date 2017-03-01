---
layout: post
title: redux 笔记
---

# redux 源码笔记

state 的改变必须经过 `store.dispatch` 方法，`dispatch` 方法会返回接收的 `action`。

## reducer

### combineReducers

```
@param {Object} reducers An object whose values correspond to different reducer functions that need to be combined into one.
@returns {Function} A reducer function that invokes every reducer inside the passed object, and builds a state object with the same shape.
```

接受多个 `reducer` 组成的 `object`，返回一个最终的 `reducer` 作为 `rootRecuer`。该 `reducer` 会将接收到的 `action` 分发给每一个接收到的 `reducer` 中。

核心代码：
```
let hasChanged = false
const nextState = {}
for (let i = 0; i < finalReducerKeys.length; i++) {
  const key = finalReducerKeys[i]
  const reducer = finalReducers[key]
  const previousStateForKey = state[key]
  const nextStateForKey = reducer(previousStateForKey, action)
  if (typeof nextStateForKey === 'undefined') {
    const errorMessage = getUndefinedStateErrorMessage(key, action)
    throw new Error(errorMessage)
  }
  nextState[key] = nextStateForKey
  hasChanged = hasChanged || nextStateForKey !== previousStateForKey
}
return hasChanged ? nextState : state
```

因为最后返回的也是一个 `reducer` ，所以 `combineReducers` 方法可以嵌套使用。

接收的参数对象的 key，对应 store state 的 key。
例如 `combineReducers({ part1: reducer1, part12: reducer2 })`，对应的 store state 形如 `{ part1: state1, part2: state2 }`

## middleware

每个 middleware 会接收两个方法 `dispatch` and `getState` 作为 named arguments。传递方法来获得 store state 来保证每个 middleware 或得的 state 为最新的。

形如 `({ dispatch, getState }) => next => action => {}`

以 `redux-thunk` 为例：

```
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

### applyMiddleware

applyMiddleware 的作用其实便是对 `store.dispatch` 的处理

```
@param {...Function} middlewares The middleware chain to be applied.
@returns {Function} A store enhancer applying the middleware.
```

首先会获取加入 middleware 前的 `dispatch` 方法（最后返回的 `dispatch` 不一样，作为 fallback dispatch），并作为每个 `middleware` 的参数。

```
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    // enhancer will always be undefined
    const store = createStore(reducer, preloadedState, enhancer)
    // as middleware's named arguments and fallback dispatch
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    // chain all middlewares together, and now every middleware now expects a next function and an action (thunk).
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

`applyMiddlewares` 这样实现的一个特性是，每一个 middleware 必须调用它的 `next` 方法，否则 chained middleware 会在此断掉。`next` 方法的参数即为下一个 middleware 的 `action`

## 有趣的设计

在 `createStore` 方法中

```
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```

在每次 `dispatch` 之前，`subscriptions` 都会被 'snapshotted'，

---

utils文件，以一个方便调试的方式打印错误信息。

打印错误信息后，会抛出并捕获一个错误信息：

```
try {
  throw new Error('...')
} catch(err) {
  // do nothing
}
```

以 Chrome 为例，如果打开 Pause on exceptions - Pause On Caught Exceptions 的话，会在此处暂停。

---

`compose` 方法

接受多个函数，并从右往左连接起来，上一个函数的值为下一个函数的参数，最右的函数可接受多个参数。

核心代码：

```
return funcs.reduce((a, b) => (...args) => a(b(...args)))
```

---
