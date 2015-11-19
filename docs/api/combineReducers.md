# `combineReducers(reducers)`

隨著你的應用程式成長的更複雜，你會想要把你的 [reducing function](../Glossary.md#reducer) 拆分成各別的 function，每一個管理獨立的某部分 [state](../Glossary.md#state)。

`combineReducers` helper function 把一個每個值都是不同的 reducing function 的物件轉換成一個你可以傳遞給 [`createStore`](createStore.md) 的單一 reducing function。

由此產生的 reducer 會呼叫每一個 child reducer，並收集它們的結果轉換成單一一個 state 物件。state 物件的形狀會符合傳遞進去的 `reducers` 的 keys。

> ##### 給 Flux 使用者的附註

> 這個 function 幫助你組織 reducers 來管理它們所擁有的一部分 state，類似於你如何擁有不同的 Flux Stores 來管理不同的 state。使用 Redux，只會有一個 store，不過 `combineReducers` 幫助你在 reducers 之間保持相同的邏輯劃分。

#### 參數

1. `reducers` (*Object*)：一個每個值都對應到不同的 reducing function 的物件，這些 reducing function 需要被合併成一個。參閱下面的附註來了解每一個被傳遞進去的 reducer 都必須遵守的一些條件。

> 早期的文件建議使用 ES6 `import * as reducers` 語法來取得 reducers 物件。這是許多困惑的來源，這就是為什麼我們現在建議從 `reducers/index.js` export 一個藉由 `combineReducers()` 獲得的單一 reducer 來取代。下面有放進來一個範例。

#### 回傳

(*Function*)：一個會呼叫在 `reducers` 物件裡面的每一個 reducer 的 reducer，並建構一個有相同形狀的 state 物件。

#### 附註

這個 function 適度的堅持並偏向幫助初學者避免常見的陷阱。這是為什麼它嘗試去強制一些如果你手動撰寫 root reducer 不需要去遵守的條件。

任何一個傳遞給 `combineReducers` 的 reducer 必須滿足這些條件：

* 對任何不認得的 action，它必須回傳給它作為第一個參數的 `state`。

* 它不應該回傳 `undefined`。非常容易因為一個提早 `return` 的 statement 而犯錯導致回傳 `undefined`，所以如果你這樣做，`combineReducers` 會拋出錯誤而不是 讓錯誤在別的地方表現出來。

* 如果傳給它的 `state` 是 `undefined`，它必須回傳初始的 state 給這個特定的 reducer。根據前一個條件，初始的 state 也不能是 `undefined`。用 ES6 optional arguments 語法來指定它很方便，不過你也可以明確地檢查第一個參數是不是 `undefined`。

雖然 `combineReducers` 會嘗試去檢查你的 reducers 是否符合這些條件，不過你應該記住它們並盡你所能的去遵守它們。

#### 範例

#### `reducers/todos.js`

```js
export default function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}
```

#### `reducers/counter.js`

```js
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

#### `reducers/index.js`

```js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
```

#### `App.js`

```js
import { createStore } from 'redux'
import reducer from './reducers/index'

let store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
```

#### 提示

* 這個 helper 只是給你一個方便！你可以撰寫你自己[用不同方式運作的](https://github.com/acdlite/reduce-reducers) `combineReducers`，或甚至手動的從 child reducers 組裝 state 物件，或就像你寫的其他 function 一樣明確的寫一個 root reducing function。

* 你可以在任何 reducer 的階層呼叫 `combineReducers`。它不需要發生在最上面。事實上你可以再次使用它把變得太複雜的 child reducers 拆分成獨立的 grandchildren，以此類推。
