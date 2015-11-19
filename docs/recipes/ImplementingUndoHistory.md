# 實作 Undo 歷史

建構 Undo 和 Redo 的功能到應用程式裡面以往需要開發者特意的努力。使用傳統的 MVC 框架時這不是一個簡單的問題，因為你需要藉由複製所有相關的 models 來持續追蹤每一個過去的 state。此外，你需要留心在 undo 的堆疊上，因為使用者造成的變動必須是可以被 undo 的。

這代表在一個 MVC 應用程式中實作 Undo 和 Redo 時常迫使你必須改寫你的應用程式的一部份以使用像是 [Command](https://en.wikipedia.org/wiki/Command_pattern) 之類特定的資料變動模式。

但是用 Redux 的話，實作 undo 歷史是一件輕而易舉的事。這有三個理由：

* 沒有許多個 models—只是一個你想要持續追蹤的 state subtree。
* state 已經是 immutable 的，而變動已經被描述為獨立的 actions，它很接近 undo 堆疊的心智模型。
* reducer 的 `(state, action) => state` signature 讓它可以很自然的實作一般的「reducer enhancers」或「higher order reducers」。它們是接收你的 reducer 並用一些額外的功能來增強它且維持一樣 signature 的 function。Undo 歷史正是一個這樣的範例。

在開始之前，請確保你已經完成了[基礎教學](../basics/README.md)並對 [reducer composition](../basics/Reducers.md) 有很好的了解。這份 recipe 將會基於描述在[基礎教學](../basics/README.md)中的範例上去建構。

在這份 recipe 的第一部份，我們將會解釋使 Undo 和 Redo 可以用一般通用的方式來實作的背後概念。

在這份 recipe 的第二部份，我們將會展示如何使用 [Redux Undo](https://github.com/omnidan/redux-undo) 套件來提供這個功能讓它立即可用。

[![demo of todos-with-undo](http://i.imgur.com/lvDFHkH.gif)](https://twitter.com/dan_abramov/status/647038407286390784)


## 了解 Undo 歷史

### 設計 State 形狀

Undo 歷史也是你應用程式的一部份 state，我們沒有理由用不同的方式處理它。無論隨著時間改變的 state 是什麼類型，當你在實做 Undo 和 Redo 時，你會想要在不同的時間點持續追蹤這個 state 的*歷史*。

例如，一個 counter 的 state 形狀看起來可能像這樣：

```js
{
  counter: 10
}
```

如果我們想要在這樣一個應用程式中實作 Undo 和 Redo，我們會需要儲存更多的 state，這樣我們才能回答下列問題：

* 有什麼東西可以 undo 或 redo？
* 現在的 state 是什麼？
* 在 undo 的堆疊中，過去 (以及未來) 的 states 是什麼？

合理建議，我們應該改變 state 的形狀來回答這些問題：

```js
{
  counter: {
    past: [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ],
    present: 10,
    future: []
  }
}
```

現在，如果使用者按下「Undo」，我們想要它變更往 past 移動：

```js
{
  counter: {
    past: [ 0, 1, 2, 3, 4, 5, 6, 7, 8 ],
    present: 9,
    future: [ 10 ]
  }
}
```

而進一步還可以：

```js
{
  counter: {
    past: [ 0, 1, 2, 3, 4, 5, 6, 7 ],
    present: 8,
    future: [ 9, 10 ]
  }
}
```

當使用者按下「Redo」，我們想要它移動一步回到 future：

```js
{
  counter: {
    past: [ 0, 1, 2, 3, 4, 5, 6, 7, 8 ],
    present: 9,
    future: [ 10 ]
  }
}
```

最後，如果使用者執行一個 action (舉例來說，對 counter 遞減)，而剛好我們在 undo 堆疊的中間時，我們必須捨棄掉已存在的 future：

```js
{
  counter: {
    past: [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ],
    present: 8,
    future: []
  }
}
```

這裡有趣的是我們想要保存數字、字串、陣列、或是物件的 undo 堆疊並不重要。它們的結構總是會一樣：

```js
{
  counter: {
    past: [ 0, 1, 2 ],
    present: 3,
    future: [ 4 ]
  }
}
```

```js
{
  todos: {
    past: [
      [],
      [ { text: 'Use Redux' } ],
      [ { text: 'Use Redux', complete: true } ]
    ],
    present: [ { text: 'Use Redux', complete: true }, { text: 'Implement Undo' } ],
    future: [
      [ { text: 'Use Redux', complete: true }, { text: 'Implement Undo', complete: true } ]
    ]
  }
}
```

一般來說，它看起來像這樣：

```js
{
  past: Array<T>,
  present: T,
  future: Array<T>
}
```

我們也可以自行決定是否要保存單一一個頂層的歷史：

```js
{
  past: [
    { counterA: 1, counterB: 1 },
    { counterA: 1, counterB: 0 },
    { counterA: 0, counterB: 0 }
  ],
  present: { counterA: 2, counterB: 1 },
  future: []
}
```

或是保存許多不同部分的歷史讓使用者可以獨立的在它們上使用 undo 或 redo actions：

```js
{
  counterA: {
    past: [ 1, 0 ],
    present: 2,
    future: []
  },
  counterB: {
    past: [ 0 ],
    present: 1,
    future: []
  }
}
```

我們將會在之後看到我們採取的方式如何讓我們選擇 Undo 和 Redo 會影響多大的部分。

### 設計演算法

不論具體的資料型別，undo 歷史的 state 形狀都一樣：

```js
{
  past: Array<T>,
  present: T,
  future: Array<T>
}
```

讓我們來探討演算法以操作在上面描述過的 state 形狀。我們可以定義兩個 actions 來在這個 state 上操作：`UNDO` 和 `REDO`。在我們的 reducer 中，我們將會做以下的步驟來處理這些 actions：

#### 處理 Undo

* 從 `past` 移除*最後一個*元素。
* 把 `present` 設成我們在前一個步驟移除的元素。
* 在 `future` *開頭*插入原本的 `present` state。

#### 處理 Redo

* 從 `future` 移除*第一個*元素。
* 把 `present` 設成我們在前一個步驟移除的元素。
* 在 `past` *尾端*插入原本的 `present` state。

#### 處理其他 Actions

* 把 `present` 插入在 `past` 尾端。
* 把 `present` 設成處理完 action 後的新 state。
* 清除 `future`。

### 第一個嘗試：寫一個 Reducer

```js
const initialState = {
  past: [],
  present: null, // (?) 我們要如何初始化 present?
  future: []
}

function undoable(state = initialState, action) {
  const { past, present, future } = state

  switch (action.type) {
    case 'UNDO':
      const previous = past[past.length - 1]
      const newPast = past.slice(0, past.length - 1)
      return {
        past: newPast,
        present: previous,
        future: [ present, ...future ]
      }
    case 'REDO':
      const next = future[0]
      const newFuture = future.slice(1)
      return {
        past: [ ...past, present ],
        present: next,
        future: newFuture
      }
    default:
      // (?) 我們要如何處理其他 actions？
      return state
  }
}
````

這份實作並不能使用，因為它忽略了三個重要的問題：

* 我們要從哪裡拿到初始的 `present` state？我們似乎無法事先知道它。
* 我們要在哪裡對外部的 actions 做出反應來把 `present` 存到 `past`？
* 我們要如何實際的把對 `present` state 的控制委託給一個自定的 reducer？

似乎 reducer 不是正確的抽象，不過我們非常接近了。

### 認識 Reducer Enhancers

你可能已經熟悉了 [higher order functions](https://en.wikipedia.org/wiki/Higher-order_function)。如果你使用 React，你可能對 [higher order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) 也很熟悉。這是同一種模式的變化，適用於 reducers。

*reducer enhancer* (或稱作 *higher order reducer*) 是一個接收 reducer，並回傳一個新的 reducer 的 function，新的 reducer 可以處理新的 actions、或保存更多的 state，並把它不了解的 actions 委託給裡面的 reducer 來控制。這不是一個新模式—技術上來說，[`combineReducers()`](../api/combineReducers.md) 也是一個 reducer enhancer，因為它接受數個 reducers 並回傳一個新的 reducer。

一個什麼事都不做的 reducer enhancer 看起來像這樣：

```js
function doNothingWith(reducer) {
  return function (state, action) {
    // 只是呼叫傳遞進去的 reducer
    return reducer(state, action)
  }
}
```

一個用來合併其他 reducers 的 reducer enhancer 看起來可能像這樣：

```js
function combineReducers(reducers) {
  return function (state = {}, action) {
    return Object.keys(reducers).reduce((nextState, key) => {
      // 用每個 reducer 管理的一部份 state 來呼叫它們
      nextState[key] = reducers[key](state[key], action)
      return nextState
    }, {})
  }
}
```

### 第二個嘗試：寫一個 Reducer Enhancer

現在我們對 reducer enhancers 有比較好的了解了，我們可以看到 `undoable` 確實應該要是個 reducer enhancer：

```js
function undoable(reducer) {
  // 用空的 action 呼叫這個 reducer 來放入初始的 state
  const initialState = {
    past: [],
    present: reducer(undefined, {}),
    future: []
  }

  // 回傳一個處理 undo 跟 redo 的 reducer
  return function (state = initialState, action) {
    const { past, present, future } = state

    switch (action.type) {
      case 'UNDO':
        const previous = past[past.length - 1]
        const newPast = past.slice(0, past.length - 1)
        return {
          past: newPast,
          present: previous,
          future: [ present, ...future ]
        }
      case 'REDO':
        const next = future[0]
        const newFuture = future.slice(1)
        return {
          past: [ ...past, present ],
          present: next,
          future: newFuture
        }
      default:
        // 委託傳進來的 reducer 來處理 action
        const newPresent = reducer(present, action)
        if (present === newPresent) {
          return state
        }
        return {
          past: [ ...past, present ],
          present: newPresent,
          future: []
        }
    }
  }
}
```

現在我們可以把任何的 reducer 包進 `undoable` reducer enhancer 來教它對 `UNDO` 和 `REDO` actions 做出反應。

```js
// 這是一個 reducer
function todos(state = [], action) {
  /* ... */
}

// 這也是一個 reducer!
const undoableTodos = undoable(todos)

import { createStore } from 'redux'
const store = createStore(undoableTodos)

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})

store.dispatch({
  type: 'ADD_TODO',
  text: 'Implement Undo'
})

store.dispatch({
  type: 'UNDO'
})
```

有一個需要注意的地方：在你要取用他的時候，你必須記得添加 `.present` 到當下的 state。你也可以分別檢查 `.past.length` 和 `.future.length` 來決定是否要啟用或禁用 Undo 和 Redo 按鈕。

你可能已經聽過 Redux 受到 [Elm 架構](https://github.com/evancz/elm-architecture-tutorial/)的影響。所以這個範例與 [elm-undo-redo 套件](http://package.elm-lang.org/packages/TheSeamau5/elm-undo-redo/2.0.0)非常相似並不令人感到訝異。

## 使用 Redux Undo

前面提供了許多有用的資訊，但是我們不能只是下載一個 library 使用它而不要自己實作 `undoable` 嗎？我們當然可以！認識 [Redux Undo](https://github.com/omnidan/redux-undo)，一個提供簡單的 Undo 和 Redo 功能給你的 Redux tree 任何部分的 library。

在這部分的 recipe 中，你將會學到如何讓 [Todo List 範例](http://rackt.github.io/redux/docs/basics/ExampleTodoList.html) 變成是 undoable 的。你可以在 [Redux 附帶的 `todos-with-undo` 範例](https://github.com/rackt/redux/tree/master/examples/todos-with-undo) 中找到這份 recipe 的完整原始碼。

### 安裝

首先，你需要執行

```
npm install --save redux-undo
```

這會安裝提供了 `undoable` reducer enhancer 的套件。

### 包裝 Reducer

你需要把你想要增強的 reducer 用 `undoable` function 包起來。例如，如果你使用 [`combineReducers()`](../api/combineReducers.md)，你的程式碼看起來可能像這樣：

#### `reducers.js`

```js
import undoable, { distinctState } from 'redux-undo'

/* ... */

const todoApp = combineReducers({
  visibilityFilter,
  todos: undoable(todos, { filter: distinctState() })
})
```

`distinctState()` filter 用來忽略沒有導致 state 改變的 actions。有[許多其他的選項](https://github.com/omnidan/redux-undo#configuration)可以用來設定你的 undoable reducer，例如設定 Undo 和 Redo actions 的 action type。

你可以在任何的 reducer composition 層級把一個或更多個 reducers 包進 `undoable` 中。我們選擇包裝 `todos` 而不是頂層 combined reducer，這樣的話對 `visibilityFilter` 的變更不會進到 undo 歷史中。

### 更新 Selectors

現在 `todos` 那部份的 state 看起來像這樣：

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: {
    past: [
      [],
      [ { text: 'Use Redux' } ],
      [ { text: 'Use Redux', complete: true } ]
    ],
    present: [ { text: 'Use Redux', complete: true }, { text: 'Implement Undo' } ],
    future: [
      [ { text: 'Use Redux', complete: true }, { text: 'Implement Undo', complete: true } ]
    ]
  }
}
```

這意味著你需要用 `state.todos.present` 來存取你的 state 而不是 `state.todos`：

#### `containers/App.js`

```js
function select(state) {
  const presentTodos = state.todos.present
  return {
    visibleTodos: selectTodos(presentTodos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  }
}
```

為了要在沒有東西可以 undo 或 redo 時讓 Undo 和 Redo 按鈕失效，你需要檢查 `past` 和 `future` 陣列是不是空的：

#### `containers/App.js`

```js
function select(state) {
  return {
    undoDisabled: state.todos.past.length === 0,
    redoDisabled: state.todos.future.length === 0,
    visibleTodos: selectTodos(state.todos.present, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}
```

### 添加按鈕

現在你只需要添加按鈕給 Undo 和 Redo actions。

首先，你需要從 `redux-undo` import `ActionCreators` 並把綁定好的版本傳遞到 `Footer` component：

#### `containers/App.js`

```js
import { ActionCreators } from 'redux-undo'

/* ... */

class App extends Component {
  render() {
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        {/* ... */}
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter => dispatch(setVisibilityFilter(nextFilter))}
          onUndo={() => dispatch(ActionCreators.undo())}
          onRedo={() => dispatch(ActionCreators.redo())}
          undoDisabled={this.props.undoDisabled}
          redoDisabled={this.props.redoDisabled} />
      </div>
    )
  }
}
```

現在你可以在 footer render 這些按鈕：

#### `components/Footer.js`

```js
export default class Footer extends Component {

  /* ... */

  renderUndo() {
    return (
      <p>
        <button onClick={this.props.onUndo} disabled={this.props.undoDisabled}>Undo</button>
        <button onClick={this.props.onRedo} disabled={this.props.redoDisabled}>Redo</button>
      </p>
    )
  }

  render() {
    return (
      <div>
        {this.renderFilters()}
        {this.renderUndo()}
      </div>
    )
  }
}
```

就這樣了！在[範例資料夾](https://github.com/rackt/redux/tree/master/examples/todos-with-undo)中執行 `npm install` 和 `npm start` 並嘗試一下！
