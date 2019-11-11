---
title: redux、react-redux
date: 2019-11-01 09:43:29
tags:
categories: React
---

# Redux

对状态变化的处理会随着项目的复杂程度提高变的很糟糕。直到你搞不清楚当一个状态改变时会都会引起什么样的变化。


## 简介：
1. redux的诞生就是为了给应用提供‘可预测化的状态管理’

2. redux会将整个应用的状态（即数据）存储到一个地方 -- store，适用createStore方法来创建一个store，该方法接收三个参数reducer、preloadedState、enhancer

3. store中保存着一棵状态树（state tree）

4. 改变状态唯一的方法就是通过store的dispatch方法，触发一个action。这个action会被对应的reducer处理，reducer为纯函数，接收state和action，返回新的state。

5. 组件通过派发（dispatch）一个行为（action）给reducer来描述发生了什么，reducer接收state和action，根据action的描述来处理对应的事情，处理完后返回新的state给store，其它的组件可以通过订阅store中state的变化来做一些事情。


## redux有三大原则：

1. 单一数据源：所有的state都以一个对象树的形式保存在一个单一的store中。

2. state只读：只能通过触发action来修改state，将所有state修改集中化处理。

3. 使用纯函数来修改：reducer接收state、action，并返回新的state。


## redux的构成

redux对外暴露五个方法，createStore、combineReducers、bindActionCreators、applyMiddleware、compose。

### createStore

createStore方法为redux的核心，该方法接收三个参数reducer(function)、preloadedState、enhancer(function),
并在函数内部创建监听事件列表(Array), 保存当前的state。
最后store返回四个方法, dispatch、subscribe、getState、replaceReducer。
在createState的最后调用一遍dispatch，传递一个Symbol作为action，触发reducer，让state的初始状态生效。

### combineReducers

该方法接收一个reducers(function)，将多个reducer合并，最终生成一个整体的reducer函数，
根据state的key去执行对应的子reducer，最后将返回结果合并成一个state tree。

### bindActionCreators

该方法接收一个Function/Object和dispatch方法，返回一个被dispatch包裹的action的方法，该方法主要是简化dispatch方法的调用。

### compose

该方法的参数为函数，compose执行返回一个函数，该函数会将所有参数函数从右向左依次执行，
且每一个参数函数执行返回的结果将作为下一个参数函数的参数。compose函数接收的参数为第一次执行的参数。
把复杂的多函数嵌套调用组合成纯粹的函数调用: fn1(fn2(fn3(fn3(...args)))) ===> compose(fn1,fn2,fn3,fn4)(...args)

### applyMiddleware

扩展Redux的一种推荐方式，可以让你包装store的dispatch方法。同时还拥有可组合性，将多个middleware组合起来形成middleware链。


## redux的原理实际上就是闭包和发布订阅模式的很好的实践。

```javascript
// 简单的发布订阅模式
let state = { count: 1 }
let listeners = [] // 订阅的事件列表

function subscribe(listener) { // 订阅事件
  listeners.push(listener)
}

function publish(count) { // 一旦发布，即调用订阅的所有事件
  state.count = count
  for (let i = 0; i < listeners.length; i++) {
    let listener = listeners[i]
    listener()
  }
}

subscribe(() => { // 订阅打印state的事件
  console.log(state)
})

// 发布状态，触发订阅的事件。
publish(1) // {count: 1}
publish(2) // {count: 2}
```


## 简单版redux实现

### createStore
该方法接收三个参数reducer（function）、preloadedState、enhancer（function），并返回四个方法dispatch、subscribe、getState、replaceReducer供store使用。

- reducer接收state、action作为参数，并返回处理后的state，该方法内需要处理state的初始状态，否则为undefined。

- preloadedState在传递reducer之前设置初始state，可以选择从服务器中获取状态或用于恢复用户历史状态。

- enhancer，store的增强器

- dispatch方法是改变redux中state的唯一方法。接收一个action作为参数，并返回一个action。

  该方法调用当前的reducer，然后将当前的state赋值为reducer处理完毕后返回的state。

  最后dispatch将store中监听的事件列表遍历执行一遍。

- subscribe方法接受一个listener(function)作为参数，该方法将listener添加到store监听的事件列表中。

  最后返回一个unsubscribe(function)，用于将listener从store监听的事件列表中移除。

- getState方法返回当前的state。

- replaceReducer方法接受一个nextReducer作为参数，将当前的reducer替换为nextReducer，

在createState的最后调用一遍dispatch，传递一个Symbol作为action，触发reducer，让state的初始状态生效。

```javascript
function createStore(reducer, preloadedState, enhancer) {
    let currentReducer = reducer
    let currentState = preloadedState;
    let currentListeners = [];
    let nextListeners = currentListeners;
    let isDispatching = false;

    if (typeof enhancer !== 'undefined') {
        return enhancer(createStore)(reducer, preloadedState)
    }

    function ensureCanMutateNextListeners() {
        if (nextListeners === currentListeners) {
          	nextListeners = currentListeners.slice()
        }
    }

    function dispatch(action) {
        try {
            isDispatching = true
            currentState = currentReducer(currentState, action)
        } finally {
          	isDispatching = false
        }
        
        let listeners = (currentListeners = nextListeners)
        for (let i = 0; i < listeners.length; i++) {
            const listener = listeners[i]
            listener()
        }

        return action;
    }

    function subscribe(listener) {
        if (!isDispatching) {
            let isSubscribe = true

            ensureCanMutateNextListeners()
            nextListeners.push(listener)
        }
        return function unsubscribe() {
            if (!isSubscribe) return;
            isSubscribe = false;

            ensureCanMutateNextListeners()
            const index = nextListeners.indexOf(listener);
            nextListeners.slice(index, 1)
            currentListeners = null
        }
    }

    function getState() {
        if (!isDispatching) {
          	return currentState;
        }
    }

    function replaceReducer(nextReducer) {
        currentReducer = nextReducer
        dispatch({ type: Symbol() })
    }

    // 用一个不匹配任何action的type, 来触发state = reducer(state, action), 获取初始值
    dispatch({ type: Symbol() })

    return {
        dispatch,
        getState,
        subscribe,
        replaceReducer,
    }
}
```

### combineReducers

```javascript
const combineReducers = reducers => {
    return (state = {}, action) => {
        return Object.keys(reducers).reduce(
            (nextState, key) => {
                nextState[key] = reducers[key](state[key], action);
                return nextState;
            },
            {} 
        );
    };
}
```

### bindActionCreators

```javascript
function bindActionCreator(actionCreator, dispatch) {
    return function() {
        return dispatch(actionCreator.apply(this, arguments))
    }
}

function bindActionCreators(actionCreators, dispatch) {
    if (typeof actionCreators === 'function') {
        return bindActionCreator(actionCreators, dispatch)
    }

    const boundActionCreators = {}
    for (const key in actionCreators) {
        const actionCreator = actionCreators[key]
        if (typeof actionCreator === 'function') {
        boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
        }
    }
    return boundActionCreators
}

// example:
    // Function
    let bindAction = bindActionCreators(a => ({ type: 'INCREMENT', params: a }), store.dispatch)
    bindAction(1)

    // Object
    let bindAction = bindActionCreators({
        INCREMENT: (a) => ({ type: 'INCREMENT', params: a }),
        DECREMENT: (a) => ({ type: 'DECREMENT', params: a })
    }, store.dispatch)
    bindAction.INCREMENT(1)
    bindAction.DECREMENT(1)
```

### compose

```javascript
function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

### applyMiddleware

每个middleware接受store的dispatch和getState函数作为命名参数，并返回一个函数。
该函数会被传入一个叫next的方法，这个next方法是下一个middleware的dispatch方法，并返回一个接收action的新函数。
这个函数可以直接调用next(action)。

```javascript
function applyMiddleware(...middlewares) {
    return createStore => (...args) => {
        const store = createStore(...args)
        let dispatch = () => {
            throw new Error(
                'Dispatching while constructing your middleware is not allowed. ' +
                'Other middleware would not be applied to this dispatch.'
            )
        }

        const middlewareAPI = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
        }
        const chain = middlewares.map(middleware => middleware(middlewareAPI))
        dispatch = compose(...chain)(store.dispatch)

        return {
            ...store,
            dispatch
        }
    }
}

// example:
const logger = ({ getState, dispacth }) => next => action => {
    console.log('老状态', getState());
    next(action);
    console.log('新状态',getState());
}
const store = applyMiddleware(logger)(createStore)(reducer);
```


***


# React-Redux

