# 疑難排解

這是一個用來分享常見問題和解決辦法的地方。
這些範例使用 React，不過如果你使用其他的東西，你應該還是會發現它們非常有用。

### 我 dispatch 一個 action 但什麼事都沒發生

有時候，你試著 dispatch 一個 action，但是你的 view 並沒有更新。為什麼會發生這種狀況呢？這有幾個可能的原因。

#### 永遠不要去變動 reducer 的參數

修改 Redux 傳遞給你的 `state` 和 `action` 很誘人。但請不要這樣做！

Redux 假設你不會在 reducer 中變動它給你的物件。**每一次，你都必須回傳新的 state 物件。**即使你沒有使用像是 [Immutable](https://facebook.github.io/immutable-js/) 之類的 library，你也需要完全避免變動。

Immutability 是讓 [react-redux](https://github.com/gaearon/react-redux) 能有效率訂閱 state 確切更新的關鍵。它也促成一些很棒的開發者體驗功能，像是用 [redux-devtools](http://github.com/gaearon/redux-devtools) 來 time travel。

例如，像是這樣的 reducer 是錯的，因為它變動了 state：

```js
function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    // 錯了！這變動了 state
    state.push({
      text: action.text,
      completed: false
    })
  case 'COMPLETE_TODO':
    // 錯了！這變動了 state[action.index]。
    state[action.index].completed = true
  }

  return state
}
```

它需要被改寫成像這樣：

```js
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      // 回傳一個新的陣列
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      // 回傳一個新的陣列
      return [
        ...state.slice(0, action.index),
        // 在變動之前先複製物件
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
    default:
      return state
  }
}
```

這樣程式碼更多了，不過這就是讓 Redux 可預測與高效能的關鍵。如果你想要減少程式碼的量，你可以使用像是 [`React.addons.update`](https://facebook.github.io/react/docs/update.html) 之類的 helper 來用一個簡潔的語法撰寫 immutable 轉換：

```js
// 使用之前：
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true
  }),
  ...state.slice(action.index + 1)
]

// 使用之後：
return update(state, {
  [action.index]: {
    completed: {
      $set: true
    }
  }
})
```

最後，要更新物件你會需要一些像是 Underscore 的 `_.extend` 的東西，或甚至更好的，一個 [`Object.assign`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 的 polyfill。

請確保你有正確地使用 `Object.assign`。例如，請從你的 reducers 回傳 `Object.assign({}, state, newData)`，而不要回傳像是 `Object.assign(state, newData)` 這樣。這樣你才不會覆寫掉前面的 `state`。

你也可以用 [Babel stage 1](http://babeljs.io/docs/usage/experimental/) 來啟用 [ES7 object spread 提案](https://github.com/sebmarkbage/ecmascript-rest-spread)：

```js
// 啟用之前：
return [
  ...state.slice(0, action.index),
  Object.assign({}, state[action.index], {
    completed: true
  }),
  ...state.slice(action.index + 1)
]

// 啟用之後：
return [
  ...state.slice(0, action.index),
  { ...state[action.index], completed: true },
  ...state.slice(action.index + 1)
]
```

需要注意的是，實驗性的語言功能有可能會變動，所以在大量的程式碼中依賴它們是不明智的。

#### 不要忘記呼叫 [`dispatch(action)`](api/Store.md#dispatch)

如果你定義了一個 action creator，呼叫它*不*會自動的 dispatch action。例如，這段程式碼什麼事都不會做：


#### `TodoActions.js`

```js
export function addTodo(text) {
  return { type: 'ADD_TODO', text }
}
```

#### `AddTodo.js`

```js
import React, { Component } from 'react'
import { addTodo } from './TodoActions'

class AddTodo extends Component {
  handleClick() {
    // 不會正常運作！
    addTodo('Fix the issue')
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    )
  }
}
```

這不會正常運作，因為你的 action creator 只是一個*回傳* action 的 function。而要由你來實際的 dispatch 它。我們不能在定義的時候把你的 action creators 綁定到一個特定的 Store 實體上，因為要在伺服器上 render 的應用程式需要對每個請求有一個獨立的 Redux store。

解決方法是在 [store](api/Store.md) 實體上呼叫 [`dispatch()`](api/Store.md#dispatch) method：

```js
handleClick() {
  // 正常運作！(不過無論如何你需要抓得到 store)
  store.dispatch(addTodo('Fix the issue'))
}
```

如果你在某個 component 層級中很深的地方，把 store 手動的傳遞下去很麻煩。這就是為什麼 [react-redux](https://github.com/gaearon/react-redux) 讓你使用一個 `connect` [higher-order component](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)，它除了會幫你訂閱 Redux store 之外，還會把 `dispatch` 注入到你的 component 的 props。

修復後的程式碼看起來像這樣：
#### `AddTodo.js`
```js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { addTodo } from './TodoActions'

class AddTodo extends Component {
  handleClick() {
    // 正常運作！
    this.props.dispatch(addTodo('Fix the issue'))
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}>
        Add
      </button>
    )
  }
}

// 除了 state 以外，`connect` 還把 `dispatch` 放到我們的 props 裡。
export default connect()(AddTodo)
```

如果你想要的話，你可以接著手動的把 `dispatch` 傳下去給其他的 components。

## 其他不能正常運作的原因

在 **#redux** [Reactiflux](http://reactiflux.com/) Discord 頻道上詢問，或是[開一個 issue](https://github.com/rackt/redux/issues)。
如果你搞清楚了，請[編輯這份文件](https://github.com/rackt/redux/edit/master/docs/Troubleshooting.md)作為好意給下一個遇到同樣問題的人。
