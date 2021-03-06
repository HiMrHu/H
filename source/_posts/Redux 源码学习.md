---
title: Redux 源码学习
date: 2018-09-11 15:12:29
tags: 源码学习
---

学习 Redux 源码

<!-- more -->

# 前置条件

注解[Redux 源码](https://github.com/iosh/Redux)

# Redux

根据[Redux 提供的 TodoList](https://github.com/reduxjs/redux/tree/master/examples/counter),思路是根据使用,反向的带入源码进行学习
这个例子中只使用了 `createStore` `dispatch` `getState`

## store

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";
import Counter from "./components/Counter";
import counter from "./reducers";

const store = createStore(counter);
const rootEl = document.getElementById("root");

const render = () =>
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({ type: "INCREMENT" })}
      onDecrement={() => store.dispatch({ type: "DECREMENT" })}
    />,
    rootEl
  );

render();
store.subscribe(render);
```

在上面的例子里, counter export default 一个函数, 即 reducer 函数, 这个函数作为参数传递给 createStore 生成 store

### createStore

redux 的[入口文件](https://github.com/iosh/Redux/blob/master/Redux/index.js),判断当前运行代码是是否为开发版本代码,以及引入并导出了所有的模块

[createStore 注释版本源码](https://github.com/iosh/Redux/blob/master/Redux/createStore.js)

createStore 函数接受 reducer 作为参数, 并且最终返回一个对象:

```javascript
 {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable

  };
```

createStore 最终返回的是上面那样一个对象,有几种方法,并且内部储存了 Reducer 的返回值作为整个状态树的 state

```jsx
const store = createStore(counter)

// store 的值变成了

{
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable

  }
```

接下来是调用 dispatch

```jsx
const render = () =>
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({ type: "INCREMENT" })}
      onDecrement={() => store.dispatch({ type: "DECREMENT" })}
    />,
    rootEl
  );

// 拿到了store 对象之后就可以调用 dispatch 方法来触发所有订阅更新
```

#### dispatch

```javascript
function dispatch(action) {
  /** 判断action是不是一个普通对象 */
  if (!isPlainObject(action)) {
    throw new Error(
      "Actions must be plain objects. " +
        "Use custom middleware for async actions."
    );
  }

  /** 判断action是否存在 type 属性 */
  if (typeof action.type === "undefined") {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
        "Have you misspelled a constant?"
    );
  }

  /** 判断是否在 dispatch 中 */
  if (isDispatching) {
    throw new Error("Reducers may not dispatch actions.");
  }

  try {
    isDispatching = true;
    /** 调用 reducer 函数并且将当前state和action传过去 */
    currentState = currentReducer(currentState, action);
  } finally {
    isDispatching = false;
  }

  /** 一串赋值, 首先将 nextListenrs 赋值给 currentListeners */
  const listeners = (currentListeners = nextListeners);
  for (let i = 0; i < listeners.length; i++) {
    /** 逐个运行订阅函数进行发布 */
    const listener = listeners[i];
    listener();
  }
  /** 返回 action  */
  return action;
}
```

dispatch 的源码就做了三件事

- 判断 action 是否合法(必须是一个普通的包含 type 属性的对象)

- 调用 reducer 函数并且传入 state 和 action 作为参数,并将 reducer 的返回值作为新的 state

- 最后逐个调用函数发布更新通知(订阅发布模式),并将 action 返回

dispatch 是发布,那么必然有一个订阅函数,用来订阅事件

#### subscribe

```jsx
const render = () =>
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({ type: "INCREMENT" })}
      onDecrement={() => store.dispatch({ type: "DECREMENT" })}
    />,
    rootEl
  );

render();
store.subscribe(render);
```

上面的代码调用 store.subscribe 并将 render 函数作为参数传递

那么 subscribe 函数的源代码为

```javascript
function subscribe(listener) {
  if (typeof listener !== "function") {
    throw new Error("Expected the listener to be a function.");
  }

  if (isDispatching) {
    throw new Error(
      "You may not call store.subscribe() while the reducer is executing. " +
        "If you would like to be notified after the store has been updated, subscribe from a " +
        "component and invoke store.getState() in the callback to access the latest state. " +
        "See https://redux.js.org/api-reference/store#subscribe(listener) for more details."
    );
  }

  let isSubscribed = true;

  ensureCanMutateNextListeners(); // 使监听器数组发生变化生成一个新的数组
  nextListeners.push(listener); // 之后 push 一个监听器

  return function unsubscribe() {
    // 之后返回一个函数用于卸载/取消监听器
    if (!isSubscribed) {
      return;
    }

    if (isDispatching) {
      throw new Error(
        "You may not unsubscribe from a store listener while the reducer is executing. " +
          "See https://redux.js.org/api-reference/store#subscribe(listener) for more details."
      );
    }

    isSubscribed = false;

    ensureCanMutateNextListeners();
    const index = nextListeners.indexOf(listener);
    nextListeners.splice(index, 1);
  };
}
```

通过分析源码可以看出 subscribe 函数 接受一个函数作为参数,之后通过构造一个通知事件列表,并将事件 push 订阅数组内
最后 subscribe 函数返回一个函数用于取消订阅

#### getState

```jsx
const render = () =>
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({ type: "INCREMENT" })}
      onDecrement={() => store.dispatch({ type: "DECREMENT" })}
    />,
    rootEl
  );
```

getState 方法的作用就非常简单了,直接返回当前的状态树,源码也非常简单.

```javascript
function getState() {
  if (isDispatching) {
    throw new Error(
      "You may not call store.getState() while the reducer is executing. " +
        "The reducer has already received the state as an argument. " +
        "Pass it down from the top reducer instead of reading it from the store."
    );
  }

  return currentState;
}
```

判断当前是否正在处理 dispatch,如果是的话抛出错误,如果不是返回当前 state 树

#### combineReducers

combineReducers 是用来组合被拆分的 reducer 将多个 reducer 组合成一个然后传递给 createStore

合并后的 reducer 可以调用各个子 reducer，并把它们返回的结果合并成一个 state 对象。
由 combineReducers() 返回的 state 对象，
会将传入的每个 reducer 返回的 state 按其传递给 combineReducers() 时对应的 key 进行命名

一般调用格式为

```javascript
import {combineReducers} from 'redux'

export default combineReducers({
  reducerName: reducer1,
  reducerName1: reducer2,
})

之后将函数返回结果传递给 createStore 生成 store
```

[带注释源码](https://github.com/iosh/Redux/blob/master/Redux/combineReducers.js)

源码中, combineReducers 在开发环境下会对每个 reducers 属性进行检查,如果其中某个 reducer 的
返回值为 undefined 那么抛出错误,以及 reducer 的值是否为函数,如果再生产环境下,会忽略所有 reducers 里面值
不是函数的 reducer, 最终构造成一个 createStore 可以使用的 reducer

而且会根据对象的 key 在 reducer 中生成对应属性名的对象, 每次逐个调用的时候都是传入 reducer 对应的部分 state

#### bindActionCreators

这个函数用的比较少

bindActionCreators(actionCreators, dispatch)

第一个参数可以使一个 actionCreatos 或者是一个 值为 actionCreatos 的对象
第二个参数就是 connect 传给的 或者 store 的 dispatch 了

例如

```javascript
function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}

bindActionCreators(addTodo, dispatch)

// 或者

const actionCreators = {
  addTodo: (text) => ({
    type: 'ADD_TODO',
    text
  })
  removeTodo: (id) => ({
    type: 'REMOVE_TODO',
    id
  })
}

bindActionCreators(actionCreators, dispatch)
```

上面两种介绍方案

源码非常简单就不过多笔记凑字数了

[带注释源码](https://github.com/iosh/Redux/blob/master/Redux/bindActionCreators.js)

#### compose

[带注释源码](https://github.com/iosh/Redux/blob/master/Redux/compose.js)

代码写的很巧妙,也没什么好说的

#### applyMiddleware

首先 createStore 判断有没有 applyMiddleware, 如果有那么就将 createStore 函数本身以及 reducer 和用于
初始化 state 的 preloadedState 作为参数传递给 applyMiddleware 的返回值

那么 applyMiddleware 的返回值是什么:

```javascript
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args);
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      );
    };

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    };
    const chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {
      ...store,
      dispatch
    };
  };
}
```

返回了一个函数,函数接受一个参数在 createStore 中是这样调用这个函数的

```javascript
enhancer(createStore)(reducer, preloadedState);
```

它将 createStore 函数本身和其他参数传递给了这个返回的函数

那么在返回头看 applyMiddleware

这时候 applyMiddleware 拿到了 createStore 函数 和函数参数,进行了一次调用调用之后拿到了 store

并且构造了一个带警告的 dispatch 函数

之后对 applyMiddleware 参数数组进行依次执行,将拿到的 getStote 方法和 构造的带错误提示的 dispatch 方法
传递给了每个中间件

之后又调用了 compose 方法,传入了所有的中间件作为参数,让中间件成为链式最后返回 store 和 他的所有方法

# 学习 React-Redux

Redux 官网提供了一个和 React 更好配合使用的例子,例子中使用了 React-redux 这个辅助库来帮助在 React 中使用 Redux

[React-Redux-TODO-LIST](https://github.com/reduxjs/redux/blob/master/examples/todos/src/index.js)

首先为了在 React 中更简单的使用 Redux , 所以 Redux 官方提供了一个 React-Redux 库,可以在 React 中高效和灵活的使用

React-Redux 库为我们提供了一个组件和一个函数

```jsx
import { Provider, connect } from "react-redux";
```

## Provider

[带注释源码](https://github.com/iosh/Redux/blob/master/React-Redux/components/Provider.js)

源码也比较简单

主要就是一个高阶组件内部使用了, `context` 这个 API (旧版), 对下属所有子组件进行通信

<Provider> 组价接受一个 store 作为属性,这个属性在内部会被赋值为组件属性, 并且设置了 childContextTypes 和 getChildContext

React 会向下自动传递参数,任何组件只要在它的子组件中,就能通过定义 contextType 来获取参数

## connect

这个组件比较复杂, 主要承担了几种职责:

- 判断传入参数是否合法
- 通过一个高阶组件,内部通过 context 订阅 store 并且判断是否发生变化来决定组件是否进行重新渲染

其他的写的比较绕,看起来不容易理解,仅仅理解了一个大概
