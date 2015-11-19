# 術語表

這是 Redux 核心詞彙的術語表，以及它們的 type signatures。這些 types 是使用 [Flow notation](http://flowtype.org/docs/quick-reference.html) 來標記。

## State

```js
type State = any
```

*State* (也稱作 *state tree*) 是一個廣義的詞彙，不過在 Redux API 中，它通常是指被 store 所管理的單一狀態值並藉由 [`getState()`](api/Store.md#getState) 來回傳。它代表 Redux 應用程式的完整狀態，通常是一個多層的巢狀物件。

慣例上，頂層 state 是個物件或一些其他 key-value collection 像是 Map，不過技術上來說它可以是任何 type。但是，你應該盡你所能的讓 state serializable。不要放任何不能輕易轉成 JSON 的東西在裡面。

## Action

```js
type Action = Object
```

*action* 是個一般的 JavaScript 物件，它代表一個改變 state 的意圖。Actions 是讓 data 進到 store 的唯一方式。任何 data，無論是從 UI 事件、網路 callbacks、或其他來源，像是 WebSockets 最後都需要作為 actions 被 dispatch。

Actions 必須有一個 `type` 屬性，它代表被執行的 action 的類型。Types 可以被定義成常數並從其他 module import。使用字串作為 `type` 會比使用 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 好，因為字串是 serializable 的。

除了 `type` 以外，action 物件的結構完全取決於你。如果你有興趣，請查看 [Flux Standard Action](https://github.com/acdlite/flux-standard-action) 上有關應該如何建構 actions 的建議。

另外請查看下面的 [async action](#async-action)。

## Reducer

```js
type Reducer<S, A> = (state: S, action: A) => S
```

*reducer* (也稱作 *reducing function*) 是一個 function，它接收累積值和一個值並回傳新的累積值。它們通常被用來把一組值 reduce down 成一個單一值。

Reducers 不是 Redux 獨有的—它們是 functional programming 裡的基礎概念。甚至大多數的非 functional languages，像是 JavaScript，都有一個內建的 reducing API。在 JavaScript 裡，那就是 [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)。

在 Redux 裡，累積值就是 state 物件，而被累積的值就是 actions。Reducers 依照先前的 state 和一個 action 來計算一個新的 state。它們必須是 *pure functions*—就是對給定的 inputs 每次會回傳完全一樣的 output 的 functions。它們也應該是沒有 side-effects 的。這就是使像是 hot reloading 和 time travel 等令人興奮的功能成真的東西。

Reducers 是 Redux 中最重要的概念。

*不要在 reducers 裡面呼叫 API*

## Dispatching Function

```js
type BaseDispatch = (a: Action) => Action
type Dispatch = (a: Action | AsyncAction) => any
```

*dispatching function* (或簡稱為 *dispatch function*) 是接收一個 action 或一個 [async action](#async-action) 的 function；接著它可能會 dispatch 一個或更多 actions 到 store，也可能不會。

我們必須區分通常的 dispatching functions 和沒有任何 middleware 的 store 實體所提供的基本 [`dispatch`](api/Store.md#dispatch) function。

基本的 dispatch function *總是*同步的把 action 發送到 store 的 reducer，伴隨著 store 回傳的先前 state，以計算一個新的 state。它預期 actions 是準備好要被 reducer 消耗的一般物件。

[Middleware](#middleware) 包裝了基本的 dispatch function。它讓 dispatch function 除了 actions 以外還可以去處理 [async actions](#async-action)。Middleware 可以在把 actions 或 async actions 傳遞到下一個 middleware 之前，以轉換、延遲、忽略、或其他別的方式處理它們。請往下看以了解更多資訊。

## Action Creator

```js
type ActionCreator = (...args: any) => Action | AsyncAction
```

*action creator* 是個產生 actions 的 function，非常的簡單。請不要混淆這兩個詞—重複一次，action 是個資訊 payload，而 action creator 是個產生 action 的 factory。

呼叫 action creator 只會產生一個 action，但是不會 dispatch 它。你需要呼叫 store 的 [`dispatch`](api/Store.md#dispatch) function 來實際地造成變化。呼叫一個 action creator 並立即 dispatch 它的結果到一個具體的 store 實體的 functions，有時我們稱 *bound action creators* 來表示。

如果 action creator 需要讀取現在的 state、執行 API 呼叫，或引起 side effect，例如 routing 轉換，它需要回傳一個 [async action](#async-action) 而不是 action。

## Async Action

```js
type AsyncAction = any
```

*async action* 是一個被送到 dispatching function 的值，但是尚未準備好要讓 reducer 消耗。它將會在被送到基本的 [`dispatch()`](api/Store.md#dispatch) function 之前，被 [middleware](#middleware) 轉換成一個 action (或一連串的 actions)。取決於你使用的 middleware，Async actions 可以有不同的種類。它們時常是一些基本的非同步類型，像是 Promise 或是 thunk，它們不會立即被傳遞到 reducer，而是在一個操作完成之後觸發 action dispatches。

## Middleware

```js
type MiddlewareAPI = { dispatch: Dispatch, getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

middleware 是一個 higher-order function，它把一個 [dispatch function](#dispatching-function) 拿去組合以回傳一個新的 dispatch function。它時常用來把 [async actions](#async-action) 轉換成 actions。

Middleware 可以藉由 function composition 來組合。有利於記錄 actions、執行有 side effects 的動作，例如：routing、或是把非同步的 API 呼叫轉換成一系列的同步 actions。

可以查看 [`applyMiddleware(...middlewares)`](./api/applyMiddleware.md) 來深入了解 middleware。

## Store

```js
type Store = {
  dispatch: Dispatch
  getState: () => State
  subscribe: (listener: () => void) => () => void
  replaceReducer: (reducer: Reducer) => void
}
```

store 是個保存應用程式 state tree 的物件。
在一個 Redux 應用程式裡，應該只有一個 store，因為 composition 發生在 reducer 層級。

- [`dispatch(action)`](api/Store.md#dispatch) 是上面描述過的基本 dispatch function。
- [`getState()`](api/Store.md#getState) 回傳現在 store 的 state。
- [`subscribe(listener)`](api/Store.md#subscribe) 註冊一個會在 state 改變時被呼叫的 function。
- [`replaceReducer(nextReducer)`](api/Store.md#replaceReducer) 可以被用來實作 hot reloading 與 code splitting。你很有可能不會使用到它們。

查看完整的 [store API 參考](api/Store.md#dispatch)來了解細節。

## Store creator

```js
type StoreCreator = (reducer: Reducer, initialState: ?State) => Store
```

store creator 是個用來建立一個 Redux store 的 function。就像 dispatching function 一樣，我們必須區分從 Redux 套件 exported 的基本的 store creator [`createStore(reducer, initialState)`](api/createStore.md)，與從 store enhancers 回傳的 store creators。

## Store enhancer

```js
type StoreEnhancer = (next: StoreCreator) => StoreCreator
```

store enhancer 是個 higher-order function，它組合 store creator 以回傳一個新的、強化的 store creator。這與 middleware 類似，它也讓你可以用組合的方式改變 store 的介面。

Store enhancers 跟 React 的 higher-order components 是大致相同的概念，後者偶爾也被稱為「component enhancers」。

因為 store 不是個物件實體，而只是一個 functions 的 collection，它的複製品可以簡單地被建立和調整而不會改變到原來的 store。在 [`compose`](api/compose.md) 文件中有一個範例展示了這個。

你很有可能永遠也不會寫到 store enhancer，但是你可能已經使用了一個由[開發工具](https://github.com/gaearon/redux-devtools)所提供的。這就是使 time travel 可以實現，但應用程式卻沒有意識到他發生的東西。有趣的是，[Redux middleware 的實作](api/applyMiddleware.md)本身就是一個 store enhancer。
