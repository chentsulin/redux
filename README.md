# <a href='http://redux.js.org'><img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60'></a>

Redux 是個給 JavaScript 應用程式所使用的可預測 state 容器。
（如果你正在尋找一個 WordPress 框架，請查看 [Redux Framework](https://reduxframework.com/)。）

他幫助你撰寫行為一致的應用程式，可以在不同的環境下執行 (客戶端、伺服器、原生應用程式)，並且易於測試。在這之上，它提供一個很棒的開發體驗，例如[把程式碼即時編輯與時間旅行除錯器結合](https://github.com/gaearon/redux-devtools)。

你可以使用 Redux 結合 [React](https://facebook.github.io/react/)，或結合其他任何的 view library。
它非常小 (2kB ，包含依賴套件)。

[![build status](https://img.shields.io/travis/reactjs/redux/master.svg?style=flat-square)](https://travis-ci.org/reactjs/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![#rackt on freenode](https://img.shields.io/badge/irc-%23rackt%20%40%20freenode-61DAFB.svg?style=flat-square)](https://webchat.freenode.net/)
[![Changelog #187](https://img.shields.io/badge/changelog-%23187-lightgrey.svg?style=flat-square)](https://changelog.com/187)

>**從 Redux 的作者學習 Redux：**
>**[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) (三十部免費影片)**

### 推薦

>[「我愛那些你在 Redux 做的東西」](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen，Flux 作者

>[「我在 FB 的內部 JS 討論群組尋求對 Redux 的評論，並獲得了普遍的好評。真的做得非常棒。」](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher，Flux 文件的作者

>[「這很酷，你藉由完全不做 Flux 來發明了一個更好的 Flux。」](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz，Cycle 作者

### 在進一步使用 Redux 之前

>**也許你也可以閱讀為什麼你可能不需要 Redux：**  
>**[「You Might Not Need Redux」](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)**

### 開發經驗

我在準備我的 React Europe 演講 [「Hot Reloading 與時間旅行」](https://www.youtube.com/watch?v=xsSnOQynTHs) 的時候撰寫了 Redux。我那時的目標是建立一個 state 管理 library，它只有最少的 API，但卻擁有完全可預測的行為，所以它可以實現 logging、hot reloading、時間旅行、universal 應用程式、記錄和重播，而不需要開發者任何其他的代價。

### 受到的影響

Redux 從 [Flux](http://facebook.github.io/flux/) 的概念發展而來，不過藉由從 [Elm](https://github.com/evancz/elm-architecture-tutorial/) 獲取線索來避免它的複雜度。
不管你以前有沒有用過它們，只需要花幾分鐘就能入門 Redux。

### 安裝

安裝穩定版本：

```
npm install --save redux
```

這裡假設你是使用 [npm](https://www.npmjs.com/) 作為你的套件管理器。
若不是的話，你可以[在 unpkg 取得這些檔案](https://unpkg.com/redux/)並下載它們，或是將套件管理器指向它們。

最常見的是人們將 Redux 作為 [CommonJS](http://webpack.github.io/docs/commonjs.html) 模組中的一個 collection 使用。當你在 [Webpack](http://webpack.github.io)、[Browserify](http://browserify.org/) 或 Node 環境中 import `redux` 時就能取得此模組。若你願意冒風險使用 [Rollup](http://rollupjs.org)，我們也同樣支援它。

如果你不想使用模組 bundler 也沒關係。`redux` npm 套件的 [`dist` 資料夾](https://unpkg.com/redux/dist/)包含了已編譯之 production 與 development 的 [UMD](https://github.com/umdjs/umd) build。你可以不透過 bundler 直接使用它們，也因此它們與許多熱門的 JavaScript 模組 loader 及環境相容。舉個例子，你可以將一個 UMD build 作為 [`<script>` 標籤](https://unpkg.com/redux/dist/redux.js)放入網頁中，或[透過 Bower 進行安裝](https://github.com/reactjs/redux/pull/1181#issuecomment-167361975)。UMD build 讓 Redux 能夠作為 `window.Redux` 全域變數進行使用。

Redux 的原始碼由 ES2015 撰寫而成，但是我們預先編譯了 CommonJS 及 UMD build 兩種 ES5 版本，讓它們可以運作於[任何現代的瀏覽器](http://caniuse.com/#feat=es5)。你不必使用 Babel 或模組 bundler 即可[開始使用 Redux](https://github.com/reactjs/redux/blob/master/examples/counter-vanilla/index.html)。

#### 補充性套件

大多數情況，你也會需要 [React 的綁定](https://github.com/reactjs/react-redux)和[開發者工具](https://github.com/gaearon/redux-devtools)。

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

請注意，這些套件不同於 Redux 自身，許多 Redux 生態系中的套件並不提供 UMD build，所以我們建議使用像是 [Webpack](http://webpack.github.io) 或 [Browserify](http://browserify.org/) 的 CommonJS 模組 bundler，以取得最舒適的開發體驗。

### 程式碼片段

你的應用程式的完整 state 被以一個 object tree 的形式儲存在單一一個的 *store* 裡面。
改變 state tree 的唯一方式是去發送一個 *action*，action 是一個描述發生什麼事的物件。
要指定 actions 要如何轉換 state tree 的話，你必須撰寫 pure *reducers*。

就這樣！

```js
import { createStore } from 'redux'

/**
 * 這是一個 reducer，一個有 (state, action) => state signature 的 pure function。
 * 它描述一個 action 如何把 state 轉換成下一個 state。
 *
 * state 的形狀取決於你：它可以是基本類型、一個陣列、一個物件，
 * 或甚至是一個 Immutable.js 資料結構。唯一重要的部分是你
 * 不應該改變 state 物件，而是當 state 變化時回傳一個新的物件。
 *
 * 在這個範例中，我們使用一個 `switch` 陳述句和字串，不過你可以使用一個 helper，
 * 來遵照一個不同的慣例 (例如 function maps)，如果它對你的專案有意義。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// 建立一個 Redux store 來掌管你的應用程式的 state。
// 它的 API 是 { subscribe, dispatch, getState }。
let store = createStore(counter)

// 在 response state 改變時，你可以使用 subscribe() 來更新你的 UI。
// 通常你會使用一個 view 綁定 library（例如：React Redux），而不是直接 subscribe()。
// 然而也可以很方便的將目前狀態儲存在 localStorage。
store.subscribe(() =>
  console.log(store.getState())
)

// 變更內部 state 的唯一方法是 dispatch 一個 action。
// actions 可以被 serialized、logged 或是儲存並在之後重播。
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

你必須指定你想要隨著被稱作 *actions* 的一般物件而發生的變更，而不是直接改變 state。接著你會寫一個被稱作 *reducer* 的特別 function，來決定每個 action 如何轉變整個應用程式的 state。

如果你以前使用 Flux，那你需要了解一個重要的差異。Redux 沒有 Dispatcher 也不支援多個 stores。反而是只有一個唯一的 store 和一個唯一的 root reducing function。當你的應用程式變大時，你會把 root reducer 拆分成比較小的獨立 reducers 來在 state tree 的不同部分上操作，而不是添加 stores。 這就像在 React 應用程式中只有一個 root component，但是他是由許多小的 components 組合而成。

這個架構用於一個計數器應用程式可能看似有點矯枉過正，不過這個模式的美妙之處就在於它如何擴展到大型且模雜的應用程式。它也啟用了非常強大的開發工具，因為它可以追蹤每一次的變更和造成變更的 action。你可以記錄使用者的 sessions 並藉由重播每個 action 來重現它們。

### 從 Redux 的作者學習它

[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) 是一個由 30 部 Dan Abramov 講述的影片組成的影片課程，他是 Redux 的作者。它被設計用來補充文件的「基礎」部分，而帶來有關 immutability、測試、Redux 最佳實踐、與搭配 React 使用 Redux 的額外洞悉。**這個課程是免費的，而且是永遠的。**

>[「在 egghead.io 上面由 @dan_abramov 出品的偉大課程 - 不只是展示了如何使用 #redux 給你看，它也展示了如何以及為什麼打造 redux！」](https://twitter.com/sandrinodm/status/670548531422326785)
>Sandrino Di Mattia

>[「鑽研過 @dan_abramov 'Getting Started with Redux' - 它藉由影片不知道簡化了多少觀念非常驚人。」](https://twitter.com/chrisdhanaraj/status/670328025553219584)
>Chris Dhanaraj

>[「@eggheadio 上面的這系列 @dan_abramov 出品的 Redux 影片非常的驚人！」](https://twitter.com/eddiezane/status/670333133242408960)
>Eddie Zaneski

>[「從名字炒作而來。而留下了堅若磐石的基礎。(感謝，@dan_abramov 和 @eggheadio 做的偉大的事！)」](https://twitter.com/danott/status/669909126554607617)
>Dan

>[「這系列 @dan_abramov 出品的 Redux 影片反覆的讓我留下深刻印象 - 打算做一些認真的重構」](https://twitter.com/gelatindesign/status/669658358643892224)
>Laurence Roberts

所以，你還在等什麼？

#### [觀看 30 部免費影片！](https://egghead.io/series/getting-started-with-redux)

如果你喜歡我的課程，請考慮藉由[購買訂閱](https://egghead.io/pricing)來支持 Egghead。訂閱者可以存取我的每一個影片中的範例的原始碼，以及無數的其他主題的進階課程，包括深入 JavaScript、React、Angular、和更多其他的。許多的 [Egghead 講師](https://egghead.io/instructors) 也是開源 library 的作者，所以購買訂閱是一個感謝他們目前所做的事的好方式。

### 文件

* [介紹](http://redux.js.org/docs/introduction/index.html)
* [基礎](http://redux.js.org/docs/basics/index.html)
* [進階](http://redux.js.org/docs/advanced/index.html)
* [Recipes](http://redux.js.org/docs/recipes/index.html)
* [疑難排解](http://redux.js.org/docs/Troubleshooting.html)
* [術語表](http://redux.js.org/docs/Glossary.html)
* [API 參考](http://redux.js.org/docs/api/index.html)

想要輸出成 PDF、ePub 和 MOBI 以方便離線閱讀的話，關於如何產生它們的說明，請參閱：[paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs)。

### 範例

* [Counter Vanilla](http://redux.js.org/docs/introduction/Examples.html#counter-vanilla) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla))
* [Counter](http://redux.js.org/docs/introduction/Examples.html#counter) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/counter))
* [Todos](http://redux.js.org/docs/introduction/Examples.html#todos) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/todos))
* [Todos with Undo](http://redux.js.org/docs/introduction/Examples.html#todos-with-undo) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo))
* [TodoMVC](http://redux.js.org/docs/introduction/Examples.html#todomvc) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/todomvc))
* [Shopping Cart](http://redux.js.org/docs/introduction/Examples.html#shopping-cart) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/shopping-cart))
* [Tree View](http://redux.js.org/docs/introduction/Examples.html#tree-view) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/tree-view))
* [Async](http://redux.js.org/docs/introduction/Examples.html#async) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/async))
* [Universal](http://redux.js.org/docs/introduction/Examples.html#universal) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/universal))
* [Real World](http://redux.js.org/docs/introduction/Examples.html#real-world) ([原始碼](https://github.com/reactjs/redux/tree/master/examples/real-world))

如果你不熟悉 NPM 生態系並在讓專案運作起來時遇到了困難，或是你不確定要在哪裡貼上上面的程式碼片段，請查看 [simplest-redux-example](https://github.com/jackielii/simplest-redux-example)，它把 Redux 和 React、Browserify 結合在一起。

### 討論

加入 [Reactiflux](http://www.reactiflux.com) Discord 社群的 [#redux](https://discord.gg/0ZcbPKXt5bZ6au5t) 頻道。

### 致謝

* [Elm 架構](https://github.com/evancz/elm-architecture-tutorial) 關於如何用 reducers 來更新 state 的偉大介紹；
* [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) 啟發我的心；
* [Developing ClojureScript with Figwheel](http://www.youtube.com/watch?v=j-kj2qwJa_E) 說服我，讓我重新評估這應該「可行」；
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) 的 Hot Module Replacement；
* [Flummox](https://github.com/acdlite/flummox) 教我如何不使用 boilerplate 和 singletons 來達成 Flux；
* [disto](https://github.com/threepointone/disto) 證明了 Stores 是 hot reloadable 的概念；
* [NuclearJS](https://github.com/optimizely/nuclear-js) 證明這個架構可以有很好的效能；
* [Om](https://github.com/omcljs/om) 推廣單一原子化 state 的想法；
* [Cycle](https://github.com/cyclejs/cycle-core) 展示 function 往往是最好的工具；
* [React](https://github.com/facebook/react) 實際的創新。

特別感謝 [Jamie Paton](http://jdpaton.github.io) 它移交了 `redux` NPM 套件名稱給我們。

### Logo

你可以在 [Github 上](https://github.com/reactjs/redux/tree/master/logo) 找到官方的 logo。

### 變更日誌

這個專案依照 [Semantic Versioning](http://semver.org/)。
每一個釋出版本都會伴隨它的遷移說明，被記錄在 Github [Releases](https://github.com/reactjs/redux/releases) 頁面上。

### 贊助者

在 Redux 的工作是[由社群出資](https://www.patreon.com/reactdx)。
遇到一些卓越的公司使這可以成真：

* [Webflow](https://github.com/webflow)
* [Ximedes](https://www.ximedes.com/)

[查看完整的 Redux 贊助者清單。](PATRONS.md)

### License

MIT
