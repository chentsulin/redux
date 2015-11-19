# API 參考

Redux 暴露出來的 API 非常少。Redux 定義一系列的介面給你去實作 (例如 [reducers](../Glossary.md#reducer)) 並提供少數的 helper functions 來把這些介面綁在一起。

這個章節把 Redux API 完整的文件化。請謹記於心，Redux 只關注管理 state。在一個實際的應用程式中，你也會想要使用像是 [react-redux](https://github.com/gaearon/react-redux) 之類的 UI 綁定。

### 頂層 Exports

* [createStore(reducer, [initialState])](createStore.md)
* [combineReducers(reducers)](combineReducers.md)
* [applyMiddleware(...middlewares)](applyMiddleware.md)
* [bindActionCreators(actionCreators, dispatch)](bindActionCreators.md)
* [compose(...functions)](compose.md)

### Store API

* [Store](Store.md)
  * [getState()](Store.md#getState)
  * [dispatch(action)](Store.md#dispatch)
  * [subscribe(listener)](Store.md#subscribe)
  * [replaceReducer(nextReducer)](Store.md#replaceReducer)

### Importing

描述在上面的每個都是頂層 export 的 function。你可以像這樣 import 它們之中任何一個：

#### ES6

```js
import { createStore } from 'redux';
```

#### ES5 (CommonJS)

```js
var createStore = require('redux').createStore;
```

#### ES5 (UMD build)

```js
var createStore = Redux.createStore;
```
