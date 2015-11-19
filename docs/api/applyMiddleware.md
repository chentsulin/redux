# `applyMiddleware(...middlewares)`

Middleware 是要用自訂的功能擴充 Redux 的推薦方式。Middleware 可以讓你為了各種理由包裝 store 的 [`dispatch`](Store.md#dispatch) method。middleware 的關鍵特色是它是可以組合的。數個 middleware 可以被結合在一起，而每一個 middleware 不需要了解在 middleware 鏈中什麼東西在它的前面或後面。

middleware 最常見的使用案例是不倚靠許多的 boilerplate 程式碼或是依賴像是 [Rx](https://github.com/Reactive-Extensions/RxJS) 之類的 library 來支援非同步的 actions。它藉由讓你除了能 dispatch 普通 action 以外還能 dispatch [async actions](../Glossary.md#async-action) 來達成。

例如，[redux-thunk](https://github.com/gaearon/redux-thunk) 讓 action creators 藉由 dispatching function 反轉控制。它們會接收 [`dispatch`](Store.md#dispatch) 作為一個參數並可能會非同步的呼叫它。這種 function 被稱作 *thunk*。另一個 middleware 的例子是 [redux-promise](https://github.com/acdlite/redux-promise)。它讓你 dispatch 一個 [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) async action，並在 Promise resolve 時 dispatch 一個一般的 action。

Middleware 沒有被內建在 [`createStore`](createStore.md) 裡面而且也不是 Redux 架構的基礎部分，不過我們認為它非常有用所以直接在核心中支援。這樣的話，有一個標準的方式在生態系裡面去擴充 [`dispatch`](Store.md#dispatch)，這樣不同的 middleware 就可以在表達力跟實用度上做競爭。

#### 參數

* `...middlewares` (*arguments*)：符合 Redux *middleware API* 的 functions。每個 middleware 會接收 [`Store`](Store.md) 的 [`dispatch`](Store.md#dispatch) 和 [`getState`](Store.md#getState) functions 作為具名參數，並回傳一個 function。這個 function 將會被給予 `next` middleware 的 dispatch method，並預期會回傳一個 `action` 的 function，可以用不同的參數呼叫 `next(action)`、或在不同的時間呼叫、或者可能完全不呼叫它。在鏈中的最後一個 middleware 將會接收實際 store 的 [`dispatch`](Store.md#dispatch) method 作為 `next` 參數，從而結束這條鏈。因此，middleware 的 signature 是 `({ getState, dispatch }) => next => action`。

#### 回傳

(*Function*) 一個啟用了給定 middleware 的 store enhancer。store enhancer 是一個需要施加到 `createStore` 的 function。它將會回傳一個不同而有啟用 middleware 的 `createStore`。

#### 範例：客製化 Logger Middleware

```js
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'

function logger({ getState }) {
  return (next) => (action) => {
    console.log('will dispatch', action)

    // 呼叫在 middleware 鏈的下一個 dispatch method。
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // 這很有可能是 action 自己，
    // 除非在鏈中更後面的 middleware 改變了它。
    return returnValue
  }
}

let createStoreWithMiddleware = applyMiddleware(logger)(createStore)
let store = createStoreWithMiddleware(todos, [ 'Use Redux' ])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware'
})
// (這幾行將會被 middleware log：)
// 將會 dispatch：{ type: 'ADD_TODO', text: 'Understand the middleware' }
// 在 dispatch 之後的 state：['Use Redux', 'Understand the middleware']
```

#### 範例：使用 Thunk Middleware 來處理 Async Actions

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from './reducers'

// applyMiddleware 會用 middleware 增強 createStore：
let createStoreWithMiddleware = applyMiddleware(thunk)(createStore)

// 我們可以就像使用「原生的」createStore 一般使用它。
let reducer = combineReducers(reducers)
let store = createStoreWithMiddleware(reducer)

function fetchSecretSauce() {
  return fetch('https://www.google.com/search?q=secret+sauce')
}

// 有一些你到目前為止所看到的一般 action creators。
// 這些 actions 回傳的東西需不需要任何 middleware 就能被 dispatch。
// 不過，他們只能表達「事實」而不是「非同步資料流」。

function makeASandwich(forPerson, secretSauce) {
  return {
    type: 'MAKE_SANDWICH',
    forPerson,
    secretSauce
  }
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: 'APOLOGIZE',
    fromPerson,
    toPerson,
    error
  }
}

function withdrawMoney(amount) {
  return {
    type: 'WITHDRAW',
    amount
  }
}

// 即使沒有 middleware，你也可以 dispatch action：
store.dispatch(withdrawMoney(100))

// 不過當你需要開始一個非同步 action 時你會怎麼做，
// 像是一個 API 呼叫，或是一個 router transition？

// 迎接 thunk。
// thunk 是一個會回傳一個 function 的 function。
// 這是一個 thunk。

function makeASandwichWithSecretSauce(forPerson) {

  // 反轉控制！
  // 回傳一個接收 `dispatch` 的 function，所以我們可以在之後進行 dispatch。
  // Thunk middleware 知道如何把 thunk async actions 轉換成 actions。

  return function (dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    )
  }
}

// Thunk middleware 讓我能 dispatch thunk async actions
// 就像它們是 actions 一樣！

store.dispatch(
  makeASandwichWithSecretSauce('Me')
)

// 它甚至會負責回傳 thunk 從 dispatch 回傳的值，
// 所以只要我有回傳 Promises 就可以串接它。

store.dispatch(
  makeASandwichWithSecretSauce('My wife')
).then(() => {
  console.log('Done!')
})

// 實際上我可以撰寫 action creators，
// 它們從其他的 action creators 來 dispatch actions 和 async actions，
// 而且我可以用 Promises 來建置我的控制流程。

function makeSandwichesForEverybody() {
  return function (dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {

      // 你不需要回傳 Promise，不過這是一個方便的慣例
      // 這樣呼叫的人總是可以在非同步 dispatch 的結果上呼叫 .then()。

      return Promise.resolve()
    }

    // 我們可以 dispatch 一般物件的 actions 以及其他的 thunks，
    // 這讓我們可以把非同步的 actions 組合在單一一個流程中。

    return dispatch(
      makeASandwichWithSecretSauce('My Grandma')
    ).then(() =>
      Promise.all([
        dispatch(makeASandwichWithSecretSauce('Me')),
        dispatch(makeASandwichWithSecretSauce('My wife'))
      ])
    ).then(() =>
      dispatch(makeASandwichWithSecretSauce('Our kids'))
    ).then(() =>
      dispatch(getState().myMoney > 42 ?
        withdrawMoney(42) :
        apologize('Me', 'The Sandwich Shop')
      )
    )
  }
}

// 這對服器端 render 非常有用，因為我可以等到
// 資料準備好，接著同步的 render 應用程式。

import { renderToString } from 'react-dom/server'

store.dispatch(
  makeSandwichesForEverybody()
).then(() =>
  response.send(renderToString(<MyApp store={store} />))
)

// 我也可以從 component dispatch 一個 thunk async action
// 任何時候它的 props 改變就會去載入缺少的資料。

import { connect } from 'react-redux'
import { Component } from 'react'

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(
      makeASandwichWithSecretSauce(this.props.forPerson)
    )
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(
        makeASandwichWithSecretSauce(nextProps.forPerson)
      )
    }
  }

  render() {
    return <p>{this.props.sandwiches.join('mustard')}</p>
  }
}

export default connect(
  state => ({
    sandwiches: state.sandwiches
  })
)(SandwichShop)
```

#### 提示

* Middleware 只包裝了 store 的 [`dispatch`](Store.md#dispatch) function。技術上來說，任何 middleware 可以做到的事情，你都可以藉由手動的包裝每一個 `dispatch` 呼叫來做到，不過在一個地方管理比較容易而且能在整個專案的層級上定義 action 轉換。

* 如果你使用其他 `applyMiddleware` 之外的 store enhancer，請確保有在組合鏈上把 `applyMiddleware` 放在它們前面，因為 middleware 有可能是非同步的。例如，它應該跑在 [redux-devtools](https://github.com/gaearon/redux-devtools) 前面，因為不這樣做的話 DevTools 無法看到 Promise middleware 以及其他等等的 middleware 發送出來的原生 actions。

* 如果你想要條件式的啟用一個 middleware，請確保只有在需要時 import 它：

  ```js
  let middleware = [ a, b ]
  if (process.env.NODE_ENV !== 'production') {
    let c = require('some-debug-middleware');
    let d = require('another-debug-middleware');
    middleware = [ ...middleware, c, d ];
  }
  const createStoreWithMiddleware = applyMiddleware(...middleware)(createStore);
  ```

  這讓建置工具可以比較簡單的去排除不需要的 modules 並減低你的建置的大小。

* 有沒有想過 `applyMiddleware` 本身是什麼呢？它理當是一個比 middleware 本身更強大的擴充機制。的確，`applyMiddleware` 是一個 Redux 最強大的擴充機制叫做 [store enhancers](../Glossary.md#store-enhancer) 的例子。很有可能你不會想要自己寫一個 store enhancer。另一個 store enhancer 的例子是 [redux-devtools](https://github.com/gaearon/redux-devtools)。Middleware 沒有 store enhancer 那麼強大，不過它寫起來比較簡單。

* Middleware 聽起來比它實際上複雜許多。要真正了解 middleware 的唯一方法就是去看現存的 middleware 是如何運作的，並試著撰寫你自己的 middleware。function 巢狀可能會很嚇人，不過你會發現實際上大部份的 middleware 只有大概 10 行左右，而巢狀與組合性是讓 middleware 系統強大的原因。
