# 伺服器 Render

伺服器端 render 最常見的使用案例是處理當使用者 (或是搜尋引擎爬蟲) 第一次送請求給我們的應用程式時的_首次 render_。當伺服器收到該請求時，它會把需要的 component(s) render 到一個 HTML 字串中，並接著把它作為回應送到客戶端。從這一個時間點開始，客戶端接管了 render 的責任。

我們將會在下面的範例使用 React，不過同樣的技術可以被使用在其他可以在伺服器 render 的 view 框架上。

### 在伺服器上的 Redux

當我們使用 Redux 來做伺服器端 render，我們必須把應用程式的 state 也在回應中一併送出去，這樣客戶端就可以使用它當作初始的 state。這很重要，因為 如果我們在產生 HTML 之前預載入了任何的資料，我們希望客戶端也能取用這份資料。否則，在客戶端產生的 markup 將不會符合伺服器的 markup，而且客戶端會必須重新再載入資料一次。

要把資料傳遞到客戶端，我們需要：

* 對每個請求建立一個全新的 Redux store 實體；
* 選擇性的 dispatch 一些 action；
* 把 state 從 store 拉出來；
* 並接著把 state 一起傳到客戶端。

在客戶端，將會建立一個新的 Redux store 並用伺服器提供的 state 來初始化。
Redux 在伺服器端的任務就**_只有_**提供**初始的 state** 給我們的應用程式。

## 設置

在接下來的 recipe 中，我們將會看一下要如何設置伺服器端 render。我們將會使用非常簡單的 [Counter app](https://github.com/rackt/redux/tree/master/examples/counter) 來教學並展示基於請求伺服器可以如何提早 render state。

### 安裝套件

在這個範例，我們將會使用 [Express](http://expressjs.com/) 作為一個簡單的 web 伺服器。我們也需要安裝 Redux 的 React 綁定，因為它們預設不包含在 Redux 裡面。

```
npm install --save express react-redux
```

## 伺服器端

下面是我們伺服器端會看起來像怎樣的概述。我們將會藉由 [app.use](http://expressjs.com/api.html#app.use) 設置一個 [Express middleware](http://expressjs.com/guide/using-middleware.html) 來處理所有進到我們伺服器的請求。如果你不熟悉 Express 或是 middleware，只要知道我們的 handleRender function 將會在每次伺服器收到請求時被呼叫就可以了。

##### `server.js`

```js
import path from 'path'
import Express from 'express'
import React from 'react'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import counterApp from './reducers'
import App from './containers/App'

const app = Express()
const port = 3000

// 這會在每次伺服器端收到請求時被呼叫
app.use(handleRender)

// 我們將會在下面的章節把這些填補起來
function handleRender(req, res) { /* ... */ }
function renderFullPage(html, initialState) { /* ... */ }

app.listen(port)
```

### 處理請求

在每個請求過來時，我們需要做的第一件事是建立一個新的 Redux store 實體。這個 store 實體的唯一目的是提供應用程式的初始 state。

在 render 的時候，我們會把 root component `<App />` 包進一個 `<Provider>` 來讓 store 可以讓在 component tree 中的所有 components 取用，正如我們在[搭配 React 運用的章節](../basics/UsageWithReact.md)所看到的。

伺服器端 render 的關鍵步驟是在我們把 component 送到客戶端_**之前**_必須把它 render 成初始的 HTML。我們使用 [ReactDOMServer.renderToString()](https://facebook.github.io/react/docs/top-level-api.html#reactdomserver.rendertostring) 來做到這一點。

接著藉由 [`store.getState()`](../api/Store.md#getState) 從我們的 Redux store 取得初始的 state。我們將會看到要如何把這個一起傳進我們的 `renderFullPage` function。

```js
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 建立一個新的 Redux store 實體
  const store = createStore(counterApp)

  // 把 component Render 成字串
  const html = renderToString(
    <Provider store={store}>
      <App />
    </Provider>
  )

  // 從我們的 Redux store 取得初始的 state
  const initialState = store.getState()

  // 把 render 完的頁面送回客戶端
  res.send(renderFullPage(html, initialState))
}
```

### 注入初始的 Component HTML 和 State

在伺服器端的最後一個步驟是把我們初始的 component HTML 和初始的 state 注入到一個要被 render 到客戶端的模板。為了把 state 傳遞下去，我們添加了一個會把 `initialState` 放進 `window.__INITIAL_STATE__` 的 `<script>` 標籤。

之後在客戶端將會可以藉由存取 `window.__INITIAL_STATE__` 來取用 `initialState`。

我們也可以藉由一個 script 標籤來引入我們給客戶端應用程式用的 bundle 檔案。這就是你的 bundle 工具針對你的客戶端進入點提供的輸出。它可以是一個靜態檔案或是去 hot reload 開發伺服器的 URL。

```js
function renderFullPage(html, initialState) {
  return `
    <!doctype html>
    <html>
      <head>
        <title>Redux Universal Example</title>
      </head>
      <body>
        <div id="app">${html}</div>
        <script>
          window.__INITIAL_STATE__ = ${JSON.stringify(initialState)}
        </script>
        <script src="/static/bundle.js"></script>
      </body>
    </html>
    `
}
```

>##### 關於 String Interpolation 語法的附註

>在上面的範例中，我們使用了 ES6 [template strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings) 語法。它讓我們撰寫多行的字串並插入他們的值，但它需要 ES6 的支援。如果你想要使用 ES6 撰寫你的 Node 程式碼，請查看 [Babel require hook](https://babeljs.io/docs/usage/require/) 的文件。或是你仍然可以撰寫 ES5 的程式碼。

## 客戶端

客戶端要做的非常簡單。我們只需要從 `window.__INITIAL_STATE__` 取得初始的 state，並把它傳遞到我們的 [`createStore()`](../api/createStore.md) function 作為初始的 state。

讓我們來看看我們新的客戶端檔案：

#### `client.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './containers/App'
import counterApp from './reducers'

// 從一個被注入進去伺服器產生的 HTML 的全域變數取得 state
const initialState = window.__INITIAL_STATE__

// 用初始的 state 來建立 Redux store
const store = createStore(counterApp, initialState)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

你可以設置你選擇的建置工具 (Webpack、Browserify、等等) 來編譯一個 bundle 的檔案到 `dist/bundle.js`。

在頁面載入時，bundle 的檔案將會被啟動並且 [`ReactDOM.render()`](https://facebook.github.io/react/docs/top-level-api.html#reactdom.render) 將會抓到從伺服器 render 的 HTML 上的 `data-react-id` 屬性。這將會把我們新啟動的 React 實體連接到在伺服器上使用的 virtual DOM。因為我們有給 Redux store 一樣的初始 state 而且使用一樣的程式碼在我們所有的 view components 上，所以結果會是一樣的 DOM。

就是這樣！這就是我們要實作伺服器端 render 所需要做的事。

但這結果非常的普通。他實際上是從動態的程式碼 render 一個靜態的 view。我們接下來需要做的是動態地建置一個初始的 state 來讓被 render 的 view 可以是動態的。

## 準備初始的 State

因為客戶端執行的是持續進行的程式碼，它可以從一個空的初始 state 開始並隨著時間推移依照需求獲得任何需要的 state。在伺服器端，render 是同步的而且 只有一次機會來 render 我們的 view。我們需要能夠在請求期間編譯初始的 state，它必須對輸入做出反應並獲得外部的 state (像是從 API 或是資料庫來的)。

### 處理請求參數

伺服器端程式碼唯一的輸入是當瀏覽器在你的應用程式中載入頁面時產生的請求。你可以選擇在啟動時選擇如何設定伺服器 (例如你是運行在一個開發或產品環境)，不過這個設定是靜態的。

請求包含了與被請求的 URL 有關的資訊，包括任何的 query 參數，它在使用一些像是 [React Router](https://github.com/rackt/react-router) 之類的東西時很有用。它也可以包涵有像是 cookies 或是授權等輸入的 headers，或是 POST body 資料。讓我們來看看我們可以如何基於 query 參數來設定初始的 counter state。

#### `server.js`

```js
import qs from 'qs' // 在檔案的最上面加上這個
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 如果有提供的話，從請求讀取 counter
  const params = qs.parse(req.query)
  const counter = parseInt(params.counter) || 0

  // 蒐集一個 initial state
  let initialState = { counter }

  // 建立一個新的 Redux store 實體
  const store = createStore(counterApp, initialState)

  // 把 component Render 成字串
  const html = renderToString(
    <Provider store={store}>
      <App />
    </Provider>
  )

  // 從我們的 Redux store 取得初始的 state
  const finalState = store.getState()

  // 把 render 完的頁面送回客戶端
  res.send(renderFullPage(html, finalState))
}
```

這份程式碼會從被傳遞進去伺服器 middleware 裡的 Express `Request` 物件進行讀取。該參數被解析成一個數字並接著設定進去初始的 state。如果你在瀏覽器訪問 [http://localhost:3000/?counter=100](http://localhost:3000/?counter=100)，你將會看到 counter 從 100 開始。在被 render 的 HTML 裡面，你會看到 counter 輸出為 100 而且 `__INITIAL_STATE__` 變數有設定 counter 在裡面。

### 非同步抓取 State

伺服器端 render 最常見的問題就是處理非同步地進來的 state。在伺服器上 Render 原本是同步的，所以需要把任何非同步的資料抓取映射到同步的操作。

要做到這個最簡單的方式是把一些 callback 傳回到你的同步程式碼裡。在這個例子中，它會是一個會參考回應物件並把被 render 的 HTML 送回到客戶端的 function。不要擔心，它沒有像它聽起來一樣那麼難。

以我們的例子來說，我們假想有一個外部的資料存儲包含了 counter 的初始值 (Counter As A Service, or CaaS)。我們將會做一個 mock 呼叫給它們並從結果來建置我們的初始 state。我們從建置 API 的呼叫開始：

#### `api/counter.js`

```js
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min)) + min
}

export function fetchCounter(callback) {
  setTimeout(() => {
    callback(getRandomInt(1, 100))
  }, 500)
}
```

再一次的，這只是 mock API，所以我們使用 `setTimeout` 來模擬一個需要花費 500 毫秒來回應的網路請求 (這應該比真實世界的 API 快許多)。我們在一個 callback 裡面非同步地傳遞一個回傳的隨機數字。如果你是使用一個基於 Promise 的 API 客戶端，那你可以在你的 `then` handler 發送這個 callback。

在伺服器端，我們簡單地把我們既有的程式碼包進 `fetchCounter` 並在 callback 中接收結果：

#### `server.js`

```js
// 把這個加到我們的 imports
import { fetchCounter } from './api/counter'
import { renderToString } from 'react-dom/server'

function handleRender(req, res) {
  // 非同步的查詢我們的 mock API
  fetchCounter(apiResult => {
    // 如果有提供的話，從請求讀取 counter
    const params = qs.parse(req.query)
    const counter = parseInt(params.counter) || apiResult || 0

    // 蒐集一個 initial state
    let initialState = { counter }

    // 建立一個新的 Redux store 實體
    const store = createStore(counterApp, initialState)

    // 把 component Render 成字串
    const html = renderToString(
      <Provider store={store}>
        <App />
      </Provider>
    )

    // 從我們的 Redux store 取得初始的 state
    const finalState = store.getState()

    // 把 render 完的頁面送回客戶端
    res.send(renderFullPage(html, finalState))
  })
}
```

因為我們在 callback 的裡面呼叫 `res.send()`，伺服器將會持續開啟連線並不會送出任何資料直到 callback 執行。你會注意到每一個伺服器的請求現在被添加一個 500ms 延遲作為我們新的 API 呼叫的結果。更進階的用法會在 API 裡優雅的處理錯誤，像是一個不好的回應或是逾時。

### 安全考量

因為我們已經導入更多依賴使用者產生的內容 (UGC) 和輸入的程式碼，我們已經增加了應用程式會被攻擊的表面積。確保你的輸入有被適當地 處理來防止像是跨站腳本 (XSS) 攻擊或是程式碼注入的東西對任何應用程式都很重要。

在我們的範例中，我們採取一個基本的方法來防護。當我們從請求獲取參數時，我們對 `counter` 參數使用 `parseInt` 來確保這個值是一個數字。如果我們沒有這樣做，你可以簡單地 藉由在請求中提供一個 script 標籤把危險的資料放進被 render 的 HTML 裡面。那可能看起來像這樣：`?counter=</script><script>doSomethingBad();</script>`

對於我們這個簡單的範例來說，強制把我們的輸入轉換成一個數字已經夠安全了。如果你是在處理更複雜的輸入，例如自由的文字，那你應該讓這個輸入通過一個適當的保護 function，例如 [validator.js](https://www.npmjs.com/package/validator)。

此外，你可以藉由對你的 state 輸出採取安全措施來添加額外的安全層。`JSON.stringify` 會受到 script 地注入。為了解決這個問題，你可以跳脫 HTML 標籤的 JSON 字串和其他危險的字元。這可以藉由在字串上做簡單的文字取代或是更複雜的 libraries 像是 [serialize-javascript](https://github.com/yahoo/serialize-javascript) 來達成。

## 下一步

你可能會想閱讀 [Async Actions](../advanced/AsyncActions.md) 來學習更多有關在 Redux 中用非同步的基礎元素像是 Promises 和 thunks 來表達非同步資料流。請記住，你在那邊學的任何東西也可以被應用在 universal rendering。

如果你使用一些像是 [React Router](https://github.com/rackt/react-router) 之類的東西，你可能也想要把你的資料抓取依賴關係表達成在你的 route handler components 上的靜態 `fetchData()` 方法。它們可以回傳 [async actions](../advanced/AsyncActions.md)，因此你的 `handleRender` function 可以把 route 匹配到幾個 route handler component class，然後 dispatch `fetchData()` 的結果給它們每一個，並只在 Promises 已經被 resolve 之後才 render。用這個方式不同的 routes 需要的特定 API 呼叫都用 route handler component 的定義放在同個地方。你也可以使用一樣的技術在客戶端來防止 router 切換頁面直到它的資料已經被載入。
