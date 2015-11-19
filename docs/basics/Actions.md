# Actions

首先，讓我們來定義一些 actions。

**Actions** 是從你的應用程式傳遞資料到你的 store 的資訊 payloads。它們是一個 store *唯一的*資訊來源。你藉由 [`store.dispatch()`](../api/Store.md#dispatch) 傳遞它們到 store。

這是一個 action 的例子，它代表添加一個新的 todo 項目：

```js
const ADD_TODO = 'ADD_TODO'
```

```js
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

Actions 是一般的 JavaScript 物件。actions 必須有一個 `type` 屬性，它代表被執行的 action 的類型。Types 通常應該被定義成字串常數。一旦你的應用程式夠大了，你能會想把它們移進一個單獨的 module。

```js
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

>##### 關於 Boilerplate 的附註

>你不需要定義 action type 常數在一個單獨的檔案，或甚至完全不需要定義它們。對小專案來說，直接使用字面字串當作 action types 可能更簡單一些。不過，在較大的程式庫中明確的定義常數有一些好處。閱讀[減少 Boilerplate](../recipes/ReducingBoilerplate.md) 來了解更多讓你的程式庫保持乾淨的實用技巧。

除了 `type` 以外，一個 action 物件的結構完全取決於你。如果你有興趣，請查看 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 上有關可以如何建構 actions 的建議。

我們再添加一個 action type 來描述一個使用者對 todo 打勾表示已完成。我們藉由 `index` 參考到特定的 todo，因為我們把他們儲存在一個陣列中。在一個實際的應用程式中，在每次新東西被創建時產生一個唯一的 ID 是比較明智的。

```js
{
  type: COMPLETE_TODO,
  index: 5
}
```

盡量讓每個 action 中傳遞的資料越少越好。舉例來說，傳遞 `index` 比傳遞整個 todo 物件來得好。

最後，我們再添加一個 action type 來改變現在顯示的 todos。

```js
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

## Action Creators

**Action creators** 就是—產生 actions 的 functions。「action」和「action creator」 這兩個詞非常容易混為一談，所以請盡你所能使用正確的術語。

在[傳統的 Flux](http://facebook.github.io/flux) 實作中，action creators 被呼叫時，通常會像這樣觸發一個 dispatch：

```js
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

相比之下，在 Redux 裡 action creators 是沒有副作用的 **pure functions**。它們簡單地回傳一個 action：

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

這使它們更具有可攜性並更容易測試。要實際地啟動一個 dispatch，必須傳遞回傳值給 `dispatch()` function：

```js
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

或者，你可以建立會自動地 dispatches 的 **bound action creator**：

```js
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))
```

現在你將可以直接去呼叫它們：

```
boundAddTodo(text)
boundCompleteTodo(index)
```

`dispatch()` function 可以藉由 [`store.dispatch()`](../api/Store.md#dispatch) 直接地從 store 取用，不過你將更可能會藉由 helper 來取用它，例如 [react-redux](http://github.com/gaearon/react-redux) 的 `connect()`。你可以使用 [`bindActionCreators()`](../api/bindActionCreators.md) 來自動綁定許多 action creators 到一個 `dispatch()` function。

## 原始碼

### `actions.js`

```js
/*
 * action types
 */

export const ADD_TODO = 'ADD_TODO'
export const COMPLETE_TODO = 'COMPLETE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * 其他常數
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

## 下一步

現在讓我們來[定義一些 reducers](Reducers.md) 以指定當你 dispatch 這些 actions 時，state 要如何更新！

>##### 給進階使用者的附註
>如果你已經熟悉基本的概念而且之前已經看完了這份教學，請不要忘記查看在[進階教學](../advanced/README.md)中的 [async actions](../advanced/AsyncActions.md)，以學習如何處理 AJAX 回應和組合 action creators 到非同步控制流程中。
