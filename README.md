# redux
```
    Redux is a predictable state container for JavaScript apps.

    Redux是JavaScript应用的一种可预测的状态容器

    曾经有人说过这样一句话 : "如果你不知道是否需要 Redux，那就是不需要它。"

    Redux 的创造者 Dan Abramov 又补充了一句 : "只有遇到 React 实在解决不了的问题，你才需要 Redux 。"

```
<br/>
<br/>

### Redux 的适用场景：多交互、多数据源
- 用户的使用方式复杂
- 不同身份的用户有不同的使用方式（比如普通用户和管理员）
- 多个用户之间可以协作
- 与服务器大量交互，或者使用了WebSocket
- View要从多个来源获取数据

### 从组件角度看，如果有以下场景，可以考虑使用 Redux。
- 某个组件的状态，需要共享
- 某个状态需要在任何地方都可以拿到
- 一个组件需要改变全局状态
- 一个组件需要改变另一个组件的状态

如果应用没那么复杂，就没必要用它。Redux 只是 Web 架构的一种解决方案


<br/>

**********

<br/>


# redux三大设计原则 
1. 单一状态树
2. 状态只读,只能由action发起变更
3. 状态只能由reducer执行变更（reducer必须是纯函数）

# 概念 store reducer action 描述
* `Store` 对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。
当前时刻的 State，可以通过`store.getState()`拿到。
    ```
        import { createStore } from 'redux';
        const store = createStore(fn);

        const state = store.getState()
    ```
* `Action` 就是 View 发出的通知，表示 State 应该要发生变化了。
Action 是一个对象。其中的`type`属性是必须的，表示 Action 的名称。Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。社区有规范可[参考](https://github.com/redux-utilities/flux-standard-action)
* Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。
    ```
        const reducer = function (state, action) {
            // ...
            return new_state;
        };
    ```
    纯函数是函数式编程的概念，必须遵守以下一些约束：
    * 不得改写参数
    * 不能调用系统 I/O 的API
    * 不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果
    ```
        // State 是一个对象
        function reducer(state, action) {
            return Object.assign({}, state, { thingToChange });
            // 或者
            return { ...state, ...newState };
        }

        // State 是一个数组
        function reducer(state, action) {
            return [...state, newItem];
        }
    ```
    整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。
    ```
        const defaultState = 0;
        const reducer = (state = defaultState, action) => {
            switch (action.type) {
                case 'ADD':
                return state + action.payload;
                default: 
                return state;
            }
        };
        const state = reducer(1, {
            type: 'ADD',
            payload: 2
        });
    ```
    上面代码中，`reducer`函数收到名为`ADD`的 Action 以后，就返回一个新的 State。实际应用中，Reducer 函数不用像上面这样手动调用。Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入`createStore`方法
    ```
        import { createStore } from 'redux';
        const store = createStore(reducer);
    ```

# createStore
    参数 (reducer, [preloadedState], [enhancer])
    reducer，
    第二个参数是它的初始状态 ，通常是服务器给的 window.STATE_FROM_SERVER
    第三个参数是enhancer 增强器,顾名思义就是增强store的功能,它的参数是创建store的函数,返回一个可以创建功能更加强大的store的函数,实际上是个高阶函数
    {
        dispatch,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable
    }

# combineReducers
`combineReducers`方法，用于 `Reducer` 的拆分。你只要定义各个子 `Reducer` 函数，然后用这个方法，将它们合成一个大的 Reducer。

总之，`combineReducers()`做的就是产生一个整体的 `Reducer` 函数。该函数根据 `State` 的 key 去执行相应的子 `Reducer`，并将返回结果合并成一个大的 `State` 对象。

```
    import { combineReducers } from 'redux';

    const chatReducer = combineReducers({
        chatLog,
        userName
    })

    export default todoApp;
```

# bindActionCreators
第一个参数是对象，第二个是dispatch
返回值 `{Function|Object}`

```
    store.dispatch(action1) 
    store.dispatch(action2)

    let wrapper =  bindActionCreators({
        click:action1,
        focus:action2
    },dispatch)
    wrapper.click('参数')
```

# applyMiddleware
作用：将所有中间件组成一个数组，依次执行。

看源码

`compose`用于形成链式中间件，从右往左执行

所有中间件被放进了一个数组`chain`，然后嵌套执行，最后执行`store.dispatch`。可以看到，中间件内部（`middlewareAPI`）可以拿到`getState`和`dispatch`这两个方法。


# 中间件 redux-thunk redux-logger
## `reudx-thunk` 解决异步
```
    import { createStore, applyMiddleware } from 'redux';
    import thunk from 'redux-thunk';
    import reducer from './reducers';

    const store = createStore(
        reducer,
        applyMiddleware(thunk)
    );
```
上面使用`redux-thunk`中间件，改造`store.dispatch`，使得后者可以接受函数作为参数。
这种解决异步的方式是 ： 写出一个返回函数的 Action Creator，然后使用`redux-thunk`中间件改造`store.dispatch`。


## `redux-logger`生成日志
生成日志中间件`logger`。然后，将它放在`applyMiddleware`方法之中，传入`createStore`方法，就完成了`store.dispatch()`的功能增强。
```
    import { applyMiddleware, createStore } from 'redux';
    import createLogger from 'redux-logger';
    const logger = createLogger();

    const store = createStore(
        reducer,
        applyMiddleware(logger)
    );
```
注意 使用中间件的时候需要注意顺序
```
    const store = createStore(
        reducer,
        applyMiddleware(thunk, promise, logger)
    );
```
比如`thunk，promise，logger`的时候。有的中间件有次序要求，使用前要查一下文档。比如，`logger`就一定要放在最后，否则输出结果会不正确。

# connect方法 （react-redux）

React-Redux 提供`connect`方法，用于从 UI 组件生成容器组件。connect的意思，就是将这两种组件连起来
connect方法的完整 API 如下。
```
    import { connect } from 'react-redux'

    const VisibleTodoList = connect(
        mapStateToProps,
        mapDispatchToProps
    )(TodoList)
```
connect方法接受两个参数：
`mapStateToProps`和`mapDispatchToProps`

`mapStateToProps`负责输入逻辑，即将`state`映射到 UI 组件的参数（`props`），`mapDispatchToProps`负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。

`mapStateToProps` 是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）state对象到（UI 组件的）props对象的映射关系。  
`mapStateToProps`(作为函数)执行后应该返回一个对象，里面的每一个键值对就是一个映射。

- 1.`mapStateToProps`是`connect`函数的第一个参数，它的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的`props`对象。
    ```
    const mapStateToProps = (state, ownProps) => {
        return {
            active: ownProps.filter === state.visibilityFilter
        }
    }
    ```
    使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发 UI 组件重新渲染。
    当然，`connect`方法可以省略`mapStateToProps`参数，这样的话 Store 的更新不会引起 UI 组件的更新。
- 2.`mapDispatchToProps`是`connect`函数的第二个参数，用来建立 UI 组件的参数到store.dispatch方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。
    如果`mapDispatchToProps`是一个函数，会得到`dispatch`和`ownProps`（容器组件的`props`对象）两个参数。
    ```
        const mapDispatchToProps = (
            dispatch,
            ownProps
        ) => {
            return {
                onClick: () => {
                dispatch({
                    type: 'SET_VISIBILITY_FILTER',
                    filter: ownProps.filter
                });
                }
            };
        }
    ```
    如果`mapDispatchToProps`是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出
    ```
        const mapDispatchToProps = {
            onClick: (filter) => {
                type: 'SET_VISIBILITY_FILTER',
                filter: filter
            };
        }
    ```

# Provider组件 （react-redux）
`connect`方法生成容器组件以后，需要让容器组件拿到`state`对象，才能生成 UI 组件的参数。
```
    import { Provider } from 'react-redux'
    import { createStore } from 'redux'
    import todoApp from './reducers'
    import App from './components/App'

    let store = createStore(todoApp);

    render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
    )
```
`Provider`在根组件外面包了一层，这样一来，`App`的所有子组件就默认都可以拿到`state`了。

