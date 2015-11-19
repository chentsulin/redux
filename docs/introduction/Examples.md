# 範例

Redux 是隨著一些範例在它的[原始碼](https://github.com/rackt/redux/tree/master/examples)中一起發佈的。
>##### 關於複製的提醒
>如果你把 Redux 範例複製到它們的目錄外面，你可以刪除它們的 `webpack.config.js` 尾端的幾行程式碼。 它們在 「You can safely delete these lines in your project.」 註解的後面。

## Counter

執行 [Counter](https://github.com/rackt/redux/tree/master/examples/counter) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/counter
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 基礎的 Redux 資料流
* 測試

## TodoMVC

執行 [TodoMVC](https://github.com/rackt/redux/tree/master/examples/todomvc) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/todomvc
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 伴隨著兩個 reducers 的 Redux 資料流
* 更新巢狀資料
* 測試

## Todos with Undo

執行 [todos-with-undo](https://github.com/rackt/redux/tree/master/examples/todos-with-undo) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/todos-with-undo
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 有兩個 reducers 的 Redux 資料流
* 在 Redux 中使用 [redux-undo](https://github.com/omnidan/redux-undo) 達成的 Undo/Redo 功能

## Async

執行 [Async](https://github.com/rackt/redux/tree/master/examples/async)

```
git clone https://github.com/rackt/redux.git

cd redux/examples/async
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 使用 [redux-thunk](https://github.com/gaearon/redux-thunk) 處理基礎的非同步 Redux 資料流
* 快取回應並在抓取資料時顯示一個 spinner
* 讓快取的資料失效

## Universal

執行 [Universal](https://github.com/rackt/redux/tree/master/examples/universal) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/universal
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 使用 Redux 和 React 來做 [Universal rendering](../recipes/ServerRendering.md)
* 基於 input 並藉由非同步查詢來提前取得 state
* 從伺服器傳遞 state 到客戶端

## Real World

執行 [Real World](https://github.com/rackt/redux/tree/master/examples/real-world) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/real-world
npm install
npm start

open http://localhost:3000/
```

它涵蓋了：

* 真實世界的非同步 Redux 資料流
* 保存 entities 在一個正規化的 entity 快取裡
* 給 API 呼叫用的客製化 middleware
* 快取回應並在抓取資料時顯示一個 spinner
* Pagination
* Routing

## Shopping Cart

執行 [Shopping Cart](https://github.com/rackt/redux/tree/master/examples/shopping-cart) 範例：

```
git clone https://github.com/rackt/redux.git

cd redux/examples/shopping-cart
npm install
npm start

open http://localhost:3000/
```

這是一個慣用的 Redux 開發模式的範例。

它涵蓋了：

* 正規化的 state
* 清楚的 entity ID 追蹤
* Reducer 組合
* 查詢跟 reducers 定義在一起
* 在失敗時 rollback 的範例
* 安全的條件式 action dispatching
* 只使用 [React Redux](https://github.com/rackt/react-redux) 來綁定 action creators
* 條件式 middleware (logging 範例)

## 更多範例

你可以在 [Awesome Redux](https://github.com/xgrommx/awesome-redux) 找到更多範例。
