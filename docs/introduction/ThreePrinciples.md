# 三大原則

Redux 可以用三個基本的原則來描述：

### 唯一真相來源

**你整個應用程式的 [state](../Glossary.md#state)，被儲存在一個樹狀物件放在唯一的 [store](../Glossary.md#store) 裡面。**

這讓建立 universal 應用程式變得更簡單，因為從伺服器來的 state 可以被 serialized 並 hydrated 進去客戶端，而不需要撰寫其他額外的程式碼。一個單一的 state tree 也讓 debug 或是調試一個應用程式更容易；它也讓你在開發期間保存應用程式的 state，以求更快的開發週期。一些傳統很難實作的功能 - 例如，復原/重做 - 可以突然變得很容易實作，如果你所有的 state 都被儲存在單一一個 single tree。

```js
console.log(store.getState())

{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

### State 是唯讀的

**改變 state 的唯一的方式是發出一個 [action](../Glossary.md#action)，也就是一個描述發生什麼事的物件。**

這能確保 views 和網路 callbacks 都不會直接寫入 state。替代的，它們表達了一個變更的意圖。因為所有的變更都是集中的，並依照嚴格的順序一個接一個的發生，沒有需要特別注意的微妙 race conditions。因為 Actions 只是普通物件，所以它們可以被記錄、serialized、儲存、並在之後為了 debugging 或測試目的而重播。

```js
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

### 變更被寫成 pure functions

**要指定 state tree 如何藉由 actions 來轉變，你必須撰寫 pure [reducers](../Glossary.md#reducer)。**

Reducers 只是 pure functions，它取得先前的 state 和一個 action，並回傳下一個 state。請記得要回傳一個新的 state 物件，而不要去改變先前的 state。你可以從單一一個 reducer 開始，而隨著你的應用程式成長，把它們拆分成比較小的 reducers 來管理 state tree 中的特定部分。因為 reducers 只是 functions，你可以控制它們被呼叫的順序、傳遞額外的資料、或甚至建立可重用的 reducers 來做一些常見的任務，例如 pagination。

```js

function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return [
        ...state.slice(0, action.index),
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
    default:
      return state
  }
}

import { combineReducers, createStore } from 'redux'
let reducer = combineReducers({ visibilityFilter, todos })
let store = createStore(reducer)
```

就這樣！現在你已經知道 Redux 都是些什麼了。
