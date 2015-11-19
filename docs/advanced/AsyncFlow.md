# 非同步資料流

不使用 [middleware](Middleware.md) 的話，Redux 的 store 只支援[同步資料流](../basics/DataFlow.md)。 這就是你使用 [`createStore()`](../api/createStore.md) 預設會拿到的。

你可以用 [`applyMiddleware()`](../api/applyMiddleware.md) 來加強 [`createStore()`](../api/createStore.md)。這不是必須的，不過這讓你[用比較方便的方式來表達 asynchronous actions](AsyncActions.md)。

非同步的 middleware 像是 [redux-thunk](https://github.com/gaearon/redux-thunk) 或是 [redux-promise](https://github.com/acdlite/redux-promise) 都包裝了 store 的 [`dispatch()`](../api/Store.md#dispatch) 方法，並允許你 dispatch 一些 actions 以外的東西，例如：functions 或 Promises。你使用的任何 middleware 可以接著解譯你 dispatch 的任何東西，並轉而傳遞 actions 到下一個鏈中的 middleware。例如，Promise middleware 可以攔截 Promises，並針對每一個 Promise 非同步的 dispatch 一對的開始/結束 actions。

當最後一個在鏈中的 middleware 要 dispatches action 時，action 必須是個一般物件。這就是[同步的 Redux 資料流](../basics/DataFlow.md)開始的地方。

請查看[非同步範例完整的原始碼](ExampleRedditAPI.md)。

## 下一步

現在你已經看過一個 middleware 在 Redux 中可以做到什麼的範例了，是時候來學習它實際上如何運作，還有你可以如何建立你自己的 middleware。前進到下一個章節有 [Middleware](Middleware.md) 相關的詳細內容。
