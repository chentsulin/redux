# `compose(...functions)`

把 functions 從右到左組合起來。

這是一個 functional programming 的 utility，並為了方便而直接被放在 Redux 裡。
你可能會想要使用它來在一行中使用幾個 [store enhancers](../Glossary.md#store-enhancer)。

#### 參數

1. (*arguments*)：要組合的 functions。每個 function 都預期會接收一個參數。它的回傳值將會作為在它左邊的 function 的參數，以此類推。

#### 回傳

(*Function*)：藉由從右到左組合給定的 functions 而獲得的最終 function。

#### 範例

這個範例展示了要如何使用 `compose` 藉由 [`applyMiddleware`](applyMiddleware.md) 與幾個來自 [redux-devtools](https://github.com/gaearon/redux-devtools) 套件的開發工具來增強一個 [store](Store.md)。

```js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from '../reducers/index'

let reducer = combineReducers(reducers)
let middleware = [ thunk ]

let finalCreateStore

// 在產品環境中，我們只想要使用 the middleware。
// 而在開發環境中，我們想要使用一些來自 redux-devtools 的 store enhancers。
// UglifyJS 會依照建置環境刪除用不到的程式碼。

if (process.env.NODE_ENV === 'production') {
  finalCreateStore = applyMiddleware(...middleware)(createStore)
} else {
  finalCreateStore = compose(
    applyMiddleware(...middleware),
    require('redux-devtools').devTools(),
    require('redux-devtools').persistState(
      window.location.href.match(/[?&]debug_session=([^&]+)\b/)
    )
  )(createStore)

  // 不使用 `compose` helper 的同等程式碼：
  //
  // finalCreateStore = applyMiddleware(middleware)(
  //   require('redux-devtools').devTools()(
  //     require('redux-devtools').persistState(
  //       window.location.href.match(/[?&]debug_session=([^&]+)\b/)
  //     )(createStore)
  //   )
  // )
}

let store = finalCreateStore(reducer)
```

#### 提示

* `compose` 做的只是讓你不需要把程式碼往右縮進，就能撰寫深度巢狀的 function 轉換。不要把它想得太複雜！
