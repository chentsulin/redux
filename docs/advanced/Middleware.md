# Middleware

你已經在 [Async Actions](../advanced/AsyncActions.md) 的範例中看過 middleware 的實際運用了。如果你使用過一些伺服器端的 libraries，像是 [Express](http://expressjs.com/) 或者是 [Koa](http://koajs.com/)，你應該對 *middleware* 的概念很熟悉。在這些框架裡，middleware 是一些你可以放到框架 接收請求和產生回應之間的程式碼。舉例來說，Express 和 Koa 的 middleware 可以添加 CORS 標頭、logging、壓縮，還有其他更多的功能。middleware 的最大特點在於它可以在鏈上組合串接。你可以在一個專案中使用多個獨立的第三方 middleware。

Redux middleware 解決了跟 Express 和 Koa middleware 不同的問題，不過概念上是類似的。**它在 dispatching action 和 action 到達 reducer 的時間點之間提供了一個第三方的擴充點。**人們可以使用 Redux middleware 來 logging、回報當機、跟非同步 API 溝通、routing，還有其他更多的功能。

這篇文章被分成深入的介紹以幫助你理解概念的部分，以及在最後面有[幾個實際的範例](#seven-examples)以展示 middleware 的力量的部分。你可能會發現當你在感到無聊與有靈感之間翻轉時，在它們之間前後切換很有幫助。

## 了解 Middleware

雖然 middleware 可以用於許多不同種類的事情，包括非同步的 API 呼叫，但是了解它從何而來真的非常重要。我們將會藉由使用 logging 和當機回報作為範例，引導你走一遍 middleware 形成的思考過程。

### 問題：Logging

Redux 的其中一個好處是可以讓 state 的改變變成可預測且透明的。當每一次 action 被 dispatched 的時候，會計算出新的 state 並儲存下來。state 無法自己改變，它只能因應特定 action 的結果而改變。

如果我們能把發生在應用程式中的每個 action，和在那之後計算出來的 state 一起  logged，那豈不是很棒？當有東西出錯了，我們可以回去看我們的 log，並找出是哪一個 action 破壞了 state。

<img src='http://i.imgur.com/BjGBlES.png' width='70%'>

我們如何運用 Redux 達到這個呢？

### 嘗試 #1：手動地 Logging

最天真的解决方案就是在你每次呼叫 [`store.dispatch(action)`](../api/Store.md#dispatch) 的時候自己 log action 和下一個 state。這不算是一個真正的解決方案，只是我們理解問題的第一步而已。

>##### 附註

>如果你是使用 [react-redux](https://github.com/gaearon/react-redux) 或是類似的綁定，你不太會在你的 components 裡面直接存取 store 實體。不過為了接下來幾個段落，就直接假設你把 store 明確地傳遞下去了。

假如說，你在建立一個 todo 的時候呼叫了這個：

```js
store.dispatch(addTodo('Use Redux'))
```

為了要 log action 和 state，你可以把它改成像是這樣：

```js
let action = addTodo('Use Redux')

console.log('dispatching', action)
store.dispatch(action)
console.log('next state', store.getState())
```

這會產生你所希望的效果，不過你不會想要每次都做這些事。

### 嘗試 #2：把 Dispatch 包起來

你可以把 logging 放進一個 function 裡：

```js
function dispatchAndLog(store, action) {
  console.log('dispatching', action)
  store.dispatch(action)
  console.log('next state', store.getState())
}
```

接著，你可以在每個地方使用它來取代 `store.dispatch()`：

```js
dispatchAndLog(store, addTodo('Use Redux'))
```

我們可以選擇在這裡結束，但是每次都要 import 一個特別的 function 還不是非常方便。

### 嘗試 #3：Monkeypatching Dispatch

那如果我們直接置換掉 store 實體上的 `dispatch` function 呢？Redux 的 store 只是一個有[幾個 methods](../api/Store.md) 的一般物件，而且我們正在寫 JavaScript，所以我們可以直接 monkeypatch `dispatch` 的實作：

```js
let next = store.dispatch
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

這已經跟我們想要的東西更接近了！無論我們在哪裡 dispatch action，都保證會被 logged 下來。Monkeypatching 的感覺總是不對，不過我們現在已經可以藉由它達成目標了。

### 問題：當機回報

那如果我們想要使用**超過一個**這樣的轉換到 `dispatch` 上呢？

浮現在我的腦海裡的另一個有用的轉換是在產品環境中回報 JavaScript 的錯誤。全域的 `window.onerror` 事件並不可靠，因為它在一些舊的瀏覽器中沒有提供堆疊資訊，而這是了解為什麼會發生錯誤的關鍵。

如果當任何時候 dispatching 一個 action 的結果丟出來錯誤，我們把它、堆疊追溯資訊、導致錯誤的 action，和當下的 state 一起傳送到一個當機回報服務，例如 [Sentry](https://getsentry.com/welcome/)，那豈不是很有用？這個方式會比較容易在開發環境中重現錯誤。

然而，讓 logging 和當機回報的部分分開是非常重要的。理想上我們希望它們是不同的模組，也可能在不同的套件中。不然，我們無法擁有一個這樣的 utilities 的生態系。(提示：我們正在慢慢的了解什麼是 middleware！)

如果 logging 和當機回報是分開的 utilities，它們看起來可能會像這樣：

```js
function patchStoreToAddLogging(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}

function patchStoreToAddCrashReporting(store) {
  let next = store.dispatch
  store.dispatch = function dispatchAndReportErrors(action) {
    try {
      return next(action)
    } catch (err) {
      console.error('Caught an exception!', err)
      Raven.captureException(err, {
        extra: {
          action,
          state: store.getState()
        }
      })
      throw err
    }
  }
}
```

如果把這些 functions 作為獨立的模組發佈，我們可以在之後使用它們來 patch 我們的 store：

```js
patchStoreToAddLogging(store)
patchStoreToAddCrashReporting(store)
```

但這依舊不夠好。

### 嘗試 #4：隱藏 Monkeypatching

Monkeypatching 是一種 hack。「置換掉任何你中意的 method」，那 API 會是怎樣呢？讓我們弄清楚它的本質。先前，我們的 functions 取代了 `store.dispatch`。那如果它們*回傳*新的 `dispatch` function 來取代呢？

```js
function logger(store) {
  let next = store.dispatch

  // 先前：
  // store.dispatch = function dispatchAndLog(action) {

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

我們可以在 Redux 裡面提供一個 helper，它會把實際運用 monkeypatching 的部分作為實作細節：

```js
function applyMiddlewareByMonkeypatching(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  // Transform dispatch function with each middleware.
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  )
}
```

我們可以像這樣使用它來啟用多個 middleware：

```js
applyMiddlewareByMonkeypatching(store, [ logger, crashReporter ])
```

不過，這仍然是 monkeypatching。
我們把它隱藏在 library 裡面並不能改變這個事實。

### 嘗試 #5：移除 Monkeypatching

為什麼我們要覆寫掉 `dispatch` 呢？當然，為了能夠在之後呼叫它，不過還有另一個理由：這樣可以讓每個 middleware 都能存取 (和呼叫) 先前被包起來的 `store.dispatch`：

```js
function logger(store) {
  // 必須指向前面的 middleware 回傳的 function：
  let next = store.dispatch

  return function dispatchAndLog(action) {
    console.log('dispatching', action)
    let result = next(action)
    console.log('next state', store.getState())
    return result
  }
}
```

串接 middleware 是不可缺少的！

在處理完第一個 middleware 之後，如果 `applyMiddlewareByMonkeypatching` 沒有立刻賦值給 `store.dispatch`，`store.dispatch` 還是會指向原本的 `dispatch` function。然後第二個 middleware 也會綁到原本的 `dispatch` function。

不過也有一個不一樣的方式可以啟用串接。這些 middleware 可以接收一個 `next()` dispatch function 作為參數而不用從 `store` 實體上取得它。

```js
function logger(store) {
  return function wrapDispatchToAddLogging(next) {
    return function dispatchAndLog(action) {
      console.log('dispatching', action)
      let result = next(action)
      console.log('next state', store.getState())
      return result
    }
  }
}
```

現在是[「我們該更深入一點」](http://knowyourmeme.com/memes/we-need-to-go-deeper)的那種時刻，所以可能會花一段時間讓它變得更合理一些。這些層層疊在一起的 function 很嚇人。ES6 arrow functions 可以讓這個 [currying](https://en.wikipedia.org/wiki/Currying) 看起來舒服一點：

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

**這就是 Redux middleware 的面貌。**

現在 middleware 接受一個 `next()` dispatch function，並回傳一個 dispatch function，它往左依序作為 middleware 的 `next()`，照此類推。如果能存取到 store 的一些 methods 像是 `getState()` 仍然非常有用，因此把 `store` 作為頂層的參數讓它依然可以使用。

### 嘗試 #6：不成熟的啟用 Middleware

作為 `applyMiddlewareByMonkeypatching()` 的替代，我們可以寫一個 `applyMiddleware()`，先取得最後完整包裝好的 `dispatch()` function，並回傳一個使用它的 store 的副本：

```js
// 警告：這是不成熟的實作！
// 這*不*是 Redux 的 API。

function applyMiddleware(store, middlewares) {
  middlewares = middlewares.slice()
  middlewares.reverse()

  let dispatch = store.dispatch
  middlewares.forEach(middleware =>
    dispatch = middleware(store)(dispatch)
  )

  return Object.assign({}, store, { dispatch })
}
```

這跟 Redux 中附帶的 [`applyMiddleware()`](../api/applyMiddleware.md) 的實作很類似，但是**有三個重要的地方不同**：

* 它只暴露了一個 [store API](../api/Store.md) 的子集給 middleware：[`dispatch(action)`](../api/Store.md#dispatch) 和 [`getState()`](../api/Store.md#getState)。

* 它用了一個很巧妙的手段來確保你是從你的 middleware 呼叫 `store.dispatch(action)` 而不是呼叫 `next(action)`，這個 action 將會實際的再次通過整個 middleware 鏈，也包括發出 action 當下的 middleware。這對非同步的 middleware 非常有用，正如我們[先前](AsyncActions.md)所看到的。

* 為了確保你只會應用 middleware 一次，它操作在 `createStore()` 上而不是在 `store` 自己上面。它的 signature 是 `(...middlewares) => (createStore) => createStore`，而不是 `(store, middlewares) => store`。

### 最後的方法

把剛剛寫的這些 middleware 再拿出來：

```js
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

以下是要如何把它運用到 Redux store 中：

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'

// applyMiddleware 接收 createStore() 並回傳
// 一個包含相容的 API 的 function。
let createStoreWithMiddleware = applyMiddleware(logger, crashReporter)(createStore)

// 像你使用 createStore() 一般使用它
let todoApp = combineReducers(reducers)
let store = createStoreWithMiddleware(todoApp)
```

就是這樣！現在任何被 dispatched 到 store 實體的 actions 都將經過 `logger` 和 `crashReporter`：

```js
// 將經過 logger 和 crashReporter 兩個 middleware！
store.dispatch(addTodo('Use Redux'))
```

## 七個範例

如果你的頭已經因為閱讀上面的章節而快燒掉了，想像一下把它寫出來會是什麼樣子。這個章節就是要讓你和我放鬆，並幫助你的齒輪繼續轉動。

下面的每一個 function 都是合格的 Redux middleware。它們不是同樣的有用，不過至少它們同樣的有趣。

```js
/**
 * 在 actions 被 dispatched 之後，Logs 所有的 actions 和 states。
 */
const logger = store => next => action => {
  console.group(action.type)
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  console.groupEnd(action.type)
  return result
}

/**
 * 在 state 被更新且 listeners 被通知之後傳送當機回報。
 */
const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}

/**
 * 用 { meta: { delay: N } } 來排程 actions 讓它延遲 N 毫秒。
 * 在這個案例中，讓 `dispatch` 回傳一個 function 來取消 timeout。
 */
const timeoutScheduler = store => next => action => {
  if (!action.meta || !action.meta.delay) {
    return next(action)
  }

  let timeoutId = setTimeout(
    () => next(action),
    action.meta.delay
  )

  return function cancel() {
    clearTimeout(timeoutId)
  }
}

/**
 * 用 { meta: { raf: true } } 來排程 actions，
 * 讓它在 rAF 迴圈中被 dispatched。在這個案例中，
 * 讓 `dispatch` 回傳一個 function 來從佇列中移除這個 action。
 */
const rafScheduler = store => next => {
  let queuedActions = []
  let frame = null

  function loop() {
    frame = null
    try {
      if (queuedActions.length) {
        next(queuedActions.shift())
      }
    } finally {
      maybeRaf()
    }
  }

  function maybeRaf() {
    if (queuedActions.length && !frame) {
      frame = requestAnimationFrame(loop)
    }
  }

  return action => {
    if (!action.meta || !action.meta.raf) {
      return next(action)
    }

    queuedActions.push(action)
    maybeRaf()

    return function cancel() {
      queuedActions = queuedActions.filter(a => a !== action)
    }
  }
}

/**
 * 讓你除了 actions 以外還可以 dispatch promises。
 * 如果這個 promise 被 resolved，它的結果將會作為 action 被 dispatched。
 * 這個 promise 會被 `dispatch` 回傳，所以呼叫者可以處理 rejection。
 */
const vanillaPromise = store => next => action => {
  if (typeof action.then !== 'function') {
    return next(action)
  }

  return Promise.resolve(action).then(store.dispatch)
}

/**
 * 讓你可以 dispatch 有 { promise } 屬性的特殊 actions。
 *
 * 這個 middleware 將會在一開始的時候 dispatch 一個 action，
 * 並在這個 `promise` resolves 的時候 dispatch 一個成功 (或失敗) 的 action。
 *
 * 為了方便，`dispatch` 將會回傳 promise 讓呼叫者可以等待。
 */
const readyStatePromise = store => next => action => {
  if (!action.promise) {
    return next(action)
  }

  function makeAction(ready, data) {
    let newAction = Object.assign({}, action, { ready }, data)
    delete newAction.promise
    return newAction
  }

  next(makeAction(false))
  return action.promise.then(
    result => next(makeAction(true, { result })),
    error => next(makeAction(true, { error }))
  )
}

/**
 * 讓你可以 dispatch 一個 function 來取代 action。
 * 這個 function 將會接收 `dispatch` 和 `getState` 作為參數。
 *
 * 對提早退出 (依照 `getState()` 的狀況)，
 * 以及非同步控制流程 (它可以 `dispatch()` 一些別的東西) 很有用。
 *
 * `dispatch` 將會回傳被 dispatched 的 function 的回傳值。
 */
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)


// 你可以使用全部！(這不意味你應該這樣做。)
let createStoreWithMiddleware = applyMiddleware(
  rafScheduler,
  timeoutScheduler,
  thunk,
  vanillaPromise,
  readyStatePromise,
  logger,
  crashReporter
)(createStore)
let todoApp = combineReducers(reducers)
let store = createStoreWithMiddleware(todoApp)
```
