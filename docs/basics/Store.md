# Store

在前面的章節，我們定義了代表實際上「發生了什麼」的 [action](Actions.md)，和依據這些 action 更新 state 的 [reducer](Reducers.md)。

**Store** 是把它們結合在一起的物件。store 有以下的責任：

* 掌管應用程式狀態；
* 允許藉由 [`getState()`](../api/Store.md#getState) 獲取 state；
* 允許藉由 [`dispatch(action)`](../api/Store.md#dispatch) 來更新 state；
* 藉由 [`subscribe(listener)`](../api/Store.md#subscribe) 註冊 listener;
* 藉由 [`subscribe(listener)`](../api/Store.md#subscribe) 回傳的 function 處理撤銷 listener。

有一點很重要需要注意，你的 Redux 應用程式中將只會有一個 store。當你想要把你的資料處理邏輯拆分時，你會使用 [reducer 合成](Reducers.md#splitting-reducers) 而不是使用許多的 store。

如果你已經有一個 reducer，要建立 store 非常簡單。在[前面的章節](Reducers.md)，我們使用 [`combineReducers()`](../api/combineReducers.md) 來把一些 reducer 合併成一個。我們現在要 import 它，並把它傳進 [`createStore()`](../api/createStore.md)。

```js
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

你可以選擇性的指定初始 state 作為第二個參數傳遞給 [`createStore()`](../api/createStore.md)。這對 hydrate 客戶端的 state 以符合運行在伺服器上的 Redux 應用程式的 state 非常有用。

```js
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## Dispatch Action

現在我們已經建立了一個 store，讓我們來驗證程式可以運作！即使沒有任何的 UI，我們也已經可以測試更新邏輯。

```js
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// 記錄初始 state
console.log(store.getState())

// 每次 state 變更，就記錄它
// 記得 subscribe() 會回傳一個用來撤銷 listener 的 function
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// Dispatch 一些 action
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// 停止監聽 state 的更新
unsubscribe()
```

你可以看到這如何造成 store 掌管的 state 改變：

<img src='http://i.imgur.com/zMMtoMz.png' width='70%'>

在我們開始撰寫 UI 之前，我們已經設定了應用程式的行為。在這個時間點，你已經可以為你的 reducer 和 action creator 撰寫測試，不過我們不會在這個教學中這樣做。你不需要 mock 任何東西，因為他們只是 [pure](../introduction/ThreePrinciples.md#changes-are-made-with-pure-functions) function。呼叫它們，然後對它們回傳的東西做出 assertion。

## 原始碼

#### `index.js`

```js
import { createStore } from 'redux'
import todoApp from './reducers'

let store = createStore(todoApp)
```

## 下一步

在為我們的 todo 應用程式建立 UI 之前，我們將會繞道看看[資料在 Redux 應用程式中如何流動](DataFlow.md)。
