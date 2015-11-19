# `createStore(reducer, [initialState])`

建立一個 Redux [store](Store.md)，它掌控應用程式的完整 state tree。
在你的應用程式中應該只有單一一個 store。

#### 參數

1. `reducer` *(Function)*：一個回傳下個 [state tree](../Glossary.md#state) 的 [reducing function](../Glossary.md#reducer)，會接收當下的 state tree 和一個要處理的 [action](../Glossary.md#action)。

2. [`initialState`] *(any)*：初始的 state。你可以選擇性的指定它來在 universal 應用程式 hydrate 從伺服器來的 state，或是用來恢復使用者先前被 serialize 的操作狀態。如果你使用 [`combineReducers`](combineReducers.md) 來產生 `reducer`，這必須是一個跟之前傳遞給它的物件有著相同形狀的 keys 的一般物件。反之，你可以自由地傳遞任何你的 `reducer` 可以了解的東西。

#### 回傳

([*`Store`*](Store.md))：掌控應用程式的完整 state 的一個物件。改變它的 state 的唯一方式是藉由 [dispatch actions](Store.md#dispatch)。你也可以[訂閱](Store.md#subscribe)它的 state 變更以更新 UI。

#### 範例

```js
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}

let store = createStore(todos, [ 'Use Redux' ])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

#### 提示

* 不要在應用程式中建立超過一個 store！作為替代，使用 [`combineReducers`](combineReducers.md) 來從多個 reducer 建立一個 root reducer。

* 要怎樣選擇 state 的格式取決於你。你可以使用一般物件或是一些像是 [Immutable](http://facebook.github.io/immutable-js/) 的東西。如果你不確定要用什麼，請從使用一般物件開始。

* 如果你的 state 是一個一般物件，請確認你從來沒有變更它！例如，不要從你的 reducers 回傳一些像是 `Object.assign(state, newData)` 的東西，應該回傳 `Object.assign({}, state, newData)`。用這個方法你不會覆寫掉先前的 `state`。如果你藉由 [Babel stage 1](http://babeljs.io/docs/usage/experimental/) 啟用 [ES7 object spread 提案](https://github.com/sebmarkbage/ecmascript-rest-spread)，你也可以寫成 `return { ...state, ...newData }`。

* 對於運行在伺服器上的 universal 應用程式，在每個請求建立一個 store 實體，因此它們彼此是隔離的。在伺服器上 render 應用程式之前，dispatch 幾個資料抓取的 actions 到 store 實體並等待它們完成。

* 當一個 store 被建立，Redux 會 dispatch 一個假的 action 到你的 reducer 來把初始的 state 填到 store。這不意味你可以直接的處理這個假 action。只要記住如果給你的 reducer 作為第一個參數的 state 是 `undefined`，那它應該回傳某種初始的 state，然後你所有的東西都會設置好。
