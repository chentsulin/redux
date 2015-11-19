# Store

store 掌控了你的應用程式的整個 [state tree](../Glossary.md#state)。
改變它裡面的 state 的唯一方法是在它上面 dispatch 一個 [action](../Glossary.md#action)。

store 不是一個 class。他只是一個有幾個 methods 在上面的物件。
要建立它，必須傳遞你的 root [reducing function](../Glossary.md#reducer) 到 [`createStore`](createStore.md)。

>##### 給 Flux 使用者的附註

>如果你是從 Flux 過來，只有一個重要的差別你必須知道。Redux 沒有 Dispatcher 也不支援多個 stores。**而只有一個使用單一 root [reducing function](../Glossary.md#reducer) 的 store。**隨著你的應用程式成長，你可以把 root reducer 拆分成較小的 reducers，獨立的在 state tree 的不同部分上操作，而不要添加 stores。你可以使用像是 [`combineReducers`](combineReducers.md) 之類的 helper 來結合它們。這相似於一個 React 應用程式中只有一個 root component，不過它是由許多小的 components 組合而成。

### Store Methods

- [`getState()`](#getState)
- [`dispatch(action)`](#dispatch)
- [`subscribe(listener)`](#subscribe)
- [`replaceReducer(nextReducer)`](#replaceReducer)

## Store Methods

### <a id='getState'></a>[`getState()`](#getState)

回傳你的應用程式當下的 state tree。
這等同於 store 的 reducer 最後一次回傳的值。

#### 回傳

*(any)*：你的應用程式當下的 state tree。

<hr>

### <a id='dispatch'></a>[`dispatch(action)`](#dispatch)

Dispatch 一個 action。這是觸發 state 變更的唯一方式。

store 的 reducing function 將會同步的用當下 [`getState()`](#getState) 的結果和給定的 `action` 來呼叫。它的回傳值將會被當作下一個 state。從現在開始 [`getState()`](#getState) 將會回傳它，而 change listeners 將會立刻被通知。

>##### 給 Flux 使用者的附註
>如果你嘗試從 [reducer](../Glossary.md#reducer) 裡面呼叫 `dispatch`，它將會拋出一個錯誤說「Reducers 不可以 dispatch actions。」這相似於在 Flux 裡面的「不可以在 dispatch 中途 dispatch」錯誤，不過不會造成那些相關的問題。在 Flux 中，當 Stores 正在處理 action 並發送更新時，是禁止 dispatch 的。不幸的，因為這樣讓它不能在 component lifecycle hooks 或是其他的好地方 dispatch actions。

>在 Redux 中，訂閱會在 root reducer 已經回傳了新的 state 之後才被呼叫，所以你*可以*在訂閱的 listeners 中 dispatch。你只被禁止在 reducers 裡面 dispatch，因為它們必須沒有 side effects。如果你想要針對一個 action 產生 side effect，做這件事的正確位置是在它的非同步 [action creator](../Glossary.md#action-creator) 裡。

#### 參數

1. `action` (*Object*<sup>†</sup>)：一個描述對你的應用程式有意義的變更的一般物件。Actions 是把資料放進 store 的唯一方法，所以任何資料，無論是從 UI 事件、網路 callbacks、或是其他來源像是 WebSockets，最後都必須做為 actions 被 dispatch。Actions 必須有一個 `type` 屬性，它代表被執行的 action 的類型。Types 可以被定義成常數並從其他 module import。使用字串作為 `type` 會比使用 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 好，因為字串是 serializable 的。除了 `type` 以外，action 物件的結構完全取決於你。如果你有興趣，請查看 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 上有關應該如何建構 actions 的建議。

#### 回傳

(Object<sup>†</sup>)：被 dispatch 的 action。

#### Notes

<sup>†</sup> 你藉由呼叫 [`createStore`](createStore.md) 所拿到的「原生」store 實作，只支援一般物件 actions 而且會立刻被送到 reducer。

但是，如果你把 [`createStore`](createStore.md) 用 [`applyMiddleware`](applyMiddleware.md) 包起來，這些 middleware 用不同的方式解釋 actions，並提供對 dispatch [async actions](../Glossary.md#async-action) 的支援。Async actions 通常是一些非同步的基礎型別，像是 Promises、Observables、或是 thunks。

Middleware 是由社群所創造且不會預設附帶在 Redux 裡。你需要明確的安裝像是 [redux-thunk](https://github.com/gaearon/redux-thunk) 或是 [redux-promise](https://github.com/acdlite/redux-promise) 之類的套件以使用它。你也可以建立自己的 middleware。
要學習如何去描述非同步的 API 呼叫、在 action creators 裡面讀取當下的 state inside、執行有 side effects 的動作、或是把它們鏈接起來按照順序執行，請查看 [`applyMiddleware`](applyMiddleware.md) 的範例。

#### 範例

```js
import { createStore } from 'redux'
let store = createStore(todos, [ 'Use Redux' ])

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

store.dispatch(addTodo('Read the docs'))
store.dispatch(addTodo('Read about the middleware'))
```

<hr>

### <a id='subscribe'></a>[`subscribe(listener)`](#subscribe)

添加一個 change listener。它將會在任何時候 action 被 dispatch 時被呼叫，而 state tree 的某部分有可能已經改變了。你可以接著呼叫 [`getState()`](#getState) 來在這個 callback 裡面讀取當下的 state。

這是一個低階 API。你大部份時候不會直接使用它，你會使用 React (或其他的) 綁定。如果你覺得這個 callback 需要使用當下的 state 當參數來呼叫，你可能會想要[把 store 轉換成 Observable 或寫一個客製化的 `observeStore` utility 來取代](https://github.com/rackt/redux/issues/303#issuecomment-125184409)。

要取消訂閱 change listener，可以呼叫 `subscribe` 回傳的 function。

#### 參數

1. `listener` (*Function*)：這個 callback 會在任何時候 action 被 dispatch 之後被呼叫，而 state tree 可能已經改變了。你可以在這個 callback 裡面呼叫 [`getState()`](#getState) 來讀取當下的 state tree。store 的 reducer 是一個 pure function，所以你可以比較 state tree 裡一些深層路徑的參考來合理預測它的值是否有改變。

##### 回傳

(*Function*)：一個可以取消訂閱 change listener 的 function。

##### 範例

```js
function select(state) {
  return state.some.deep.property
}

let currentValue
function handleChange() {
  let previousValue = currentValue
  currentValue = select(store.getState())

  if (previousValue !== currentValue) {
    console.log('Some deep nested property changed from', previousValue, 'to', currentValue)
  }
}

let unsubscribe = store.subscribe(handleChange)
handleChange()
```

<hr>

### <a id='replaceReducer'></a>[`replaceReducer(nextReducer)`](#replaceReducer)

置換 store 當下用來計算 state 使用的 reducer。

這是一個進階的 API。如果你的應用程式要實作 code splitting 你可能會需要這個，因為你想要動態的載入一些 reducers。如果你要實作一個 Redux 的 hot reloading 機制，那你也可能需要這個。

#### 參數

1. `reducer` (*Function*)：給 store 接下來使用的 reducer。
