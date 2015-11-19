# [Redux](http://rackt.github.io/redux)

Redux 是個給 JavaScript 應用程式所使用的可預測 state 容器。

他幫助你撰寫行為一致的應用程式，可以在不同的環境下執行 (客戶端、伺服器、原生應用程式)，並且易於測試。在這之上，它提供一個很棒的開發體驗，例如[把程式碼即時編輯與時間旅行除錯器結合](https://github.com/gaearon/redux-devtools)。

你可以使用 Redux 結合 [React](https://facebook.github.io/react/)，或結合其他任何的 view library。
它非常小 (2kB) 並且沒有任何依賴套件。

[![build status](https://img.shields.io/travis/rackt/redux/master.svg?style=flat-square)](https://travis-ci.org/rackt/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![#rackt on freenode](https://img.shields.io/badge/irc-%23rackt%20%40%20freenode-61DAFB.svg?style=flat-square)](https://webchat.freenode.net/)


### 推薦

>[「我愛那些你在 Redux 做的東西」](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen，Flux 作者

>[「我在 FB 的內部 JS 討論群組尋求對 Redux 的評論，並獲得了普遍的好評。真的做得非常棒。」](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher，Flux 文件的作者

>[「這很酷，你藉由完全不做 Flux 來發明了一個更好的 Flux。」](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz，Cycle 作者

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

大多數情況，你也會需要 [React 的綁定](https://github.com/rackt/react-redux)和[開發者工具](https://github.com/gaearon/redux-devtools)。

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

這裡假設你是使用 [npm](https://www.npmjs.com/) 套件管理與一個模組 bundler，像是 [Webpack](http://webpack.github.io) 或是 [Browserify](http://browserify.org/) 來使用 [CommonJS 模組](http://webpack.github.io/docs/commonjs.html)。

如果你還沒有使用 [npm](https://www.npmjs.com/) 或任何一個現代的模組 bundler，而且寧願使用可以讓 `Redux` 作為一個全域物件使用的單檔 [UMD](https://github.com/umdjs/umd) 編譯，你可以從 [cdnjs](https://cdnjs.com/libraries/redux) 取得一個預先編譯好的版本。對於任何重要的應用程式，我們*不*建議使用這個方法，因為大部份與 Redux 互補的 libraries 都只能在 [npm](https://www.npmjs.com/) 上取得。

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

// 你可以手動的去訂閱更新，或是使用跟你的 view layer 之間的綁定。
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

### 文件

* [介紹](http://rackt.github.io/redux/docs/introduction/index.html)
* [基礎](http://rackt.github.io/redux/docs/basics/index.html)
* [進階](http://rackt.github.io/redux/docs/advanced/index.html)
* [Recipes](http://rackt.github.io/redux/docs/recipes/index.html)
* [疑難排解](http://rackt.github.io/redux/docs/Troubleshooting.html)
* [術語表](http://rackt.github.io/redux/docs/Glossary.html)
* [API 參考](http://rackt.github.io/redux/docs/api/index.html)

想要輸出成 PDF、ePub 和 MOBI 以方便離線閱讀的話，關於如何產生它們的說明，請參閱：[paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs)。

### 範例

* [Counter](http://rackt.github.io/redux/docs/introduction/Examples.html#counter) ([原始碼](https://github.com/rackt/redux/tree/master/examples/counter))
* [TodoMVC](http://rackt.github.io/redux/docs/introduction/Examples.html#todomvc) ([原始碼](https://github.com/rackt/redux/tree/master/examples/todomvc))
* [Todos with Undo](http://rackt.github.io/redux/docs/introduction/Examples.html#todos-with-undo) ([原始碼](https://github.com/rackt/redux/tree/master/examples/todos-with-undo))
* [Async](http://rackt.github.io/redux/docs/introduction/Examples.html#async) ([原始碼](https://github.com/rackt/redux/tree/master/examples/async))
* [Universal](http://rackt.github.io/redux/docs/introduction/Examples.html#universal) ([原始碼](https://github.com/rackt/redux/tree/master/examples/universal))
* [Real World](http://rackt.github.io/redux/docs/introduction/Examples.html#real-world) ([原始碼](https://github.com/rackt/redux/tree/master/examples/real-world))
* [Shopping Cart](http://rackt.github.io/redux/docs/introduction/Examples.html#shopping-cart) ([原始碼](https://github.com/rackt/redux/tree/master/examples/shopping-cart))

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

### 變更日誌

這個專案依照 [Semantic Versioning](http://semver.org/)。
每一個釋出版本都會伴隨它的遷移說明，被記錄在 Github [Releases](https://github.com/rackt/redux/releases) 頁面上。

### 贊助者

在 Redux 的工作是[由社群出資](https://www.patreon.com/reactdx)。
遇到一些卓越的公司使這可以成真：

* [Webflow](https://webflow.com/)
* [Chess iX](http://www.chess-ix.com/)

[查看完整的 Redux 贊助者清單。](PATRONS.md)

### License

MIT
