# Redux 常見問答集

## 常見問答內容列表

- **一般常見問題**
  - [我應該什麼時候使用 Redux？](#general-when-to-use)
  - [Redux 只能用在 React 上嗎？](#general-only-react)
  - [我需要特定的建構工具才能使用 Redux 嗎？](#general-build-tools)
- **Reducer**
  - [我要如何在兩個 reducer 之間共用 state？我一定要使用 combineReducers 嗎？](#reducers-share-state)
  - [我一定要使用 switch 條件式來處理 action 嗎？](#reducers-use-switch)
- **組織 State**
  - [我一定要把我所有 state 放入 Redux 嗎？我應該使用 React 的 setState() 嗎？](#organizing-state-only-redux-state)
  - [我可以在 store 的 state 內放置 function、promise、或其他無法 serialize 的東西嗎？](#organizing-state-non-serializable)
  - [我要如何在我的 state 組織巢狀和重複的資料？](#organizing-state-nested-data)
- **設置 Store**
  - [我可以或是說我應該建立多個 store 嗎？我可以直接匯入我的 store，然後在我的 component 中使用嗎？](#store-setup-multiple-stores)
  - [是不是可以有一個以上的 middleware 鏈在我的 store enhancer 中？在 middleware function 的 next 和 dispatch 的區別是什麼？](#store-setup-middleware-chains)
  - [我要如何只訂閱部份的 state？我可以取得被 dispatch 的 action 當做訂閱的一部分嗎？](#store-setup-subscriptions)
- **Action**
  - [為什麼 type 要是一個字串，或是至少可以 serialize 的？為什麼我的 action type 應該要是常數？](#actions-string-constants)
  - [reducer 和 action 之間一定是一對一互相對應嗎？](#actions-reducer-mappings)
  - [我如何表達像是 AJAX 呼叫之類的「side effect」？為什麼我們需要像是「action creator」、「thunk」、以及「middleware」來執行非同步的行為？](#actions-side-effects)
  - [我應該從 action creator 連續 dispatch 多個 action 嗎？](#actions-multiple-actions)
- **程式碼結構**
  - [我的檔案結構應該長什麼樣子？在我的專案中，我應該如何組織我的 action creator 和 reducer？我的 selector 應該放哪？](#structure-file-structure)
  - [我應該如何將我的 reducer 和 action creator 之間的邏輯拆開？我的「商業邏輯」應該放在哪裡？](#structure-business-logic)
- **效能**
  - [依據效能和架構方面來說，Redux 能「scale」的多好？](#performance-scaling)
  - [針對每個 action 呼叫「我所有的 reducer」不會很慢嗎？](#performance-all-reducers)
  - [我需要在我的 reducer 深層複製我的 state 嗎？複製 state 不會導致變慢嗎？](#performance-clone-state)
  - [我如何減少 store 更新事件的次數？](#performance-update-events)
  - [「一個 state tree」會造成記憶體上的問題嗎？dispatch 許多 action 會佔用記憶體嗎？](#performance-state-memory)
- **React Redux**
  - [為什麼我的 component 不會重新 render，或者是執行 mapStateToProps？](#react-not-rerendering)
  - [為什麼我的 component 重新 render 過於頻繁？](#react-rendering-too-often)
  - [我如何讓我的 mapStateToProps 變快？](#react-mapstate-speed)
  - [為什麼我在 connect 的 component 中沒有 this.props.dispatch 可以使用？](#react-props-dispatch)
  - [我應該只 connect 我的頂層 component，或者我可以在我的 tree 中 connect 更多的 component？](#react-multiple-components)
- **雜項**
  - [有沒有任何更大的、「實際的」Redux 專案？](#miscellaneous-real-projects)
  - [我該如何在 Redux 中實作認證？](#miscellaneous-authentication)


## 一般常見問題

<a id="general-when-to-use"></a>
### 我應該什麼時候使用 Redux？

早期的 React contributor 之一，Pete Hunt 說：

> 當你需要 Flux 的時候，你會知道。如果你不確定你是否需要，那就是不需要了。

同樣的，Redux 的作者之一 Dan Abramov 這麼說：

> 我會想要這樣說：不要使用 Redux，直到你用原生的 React 出現了問題。

一般來說，在當你有合理的資料量會隨著時間的推移而改變，你需要唯一的資料來源，而且你發現像是把所有東西保存在頂層 React component 的 state 之類的方法不再是足夠的時候使用 Redux。

#### 更多資訊

**文件**
- [簡介：動機](introduction/Motivation.md)

**討論**
- [React How-To](https://github.com/petehunt/react-howto)
- [Twitter - 不要使用 Redux，直到...](https://twitter.com/dan_abramov/status/699241546248536064)
- [Flux 的案例](https://medium.com/swlh/the-case-for-flux-379b7d1982c6)
- [Stack Overflow - 為什麼 Redux 會超越 Facebook 的 Flux？ Flux?](http://stackoverflow.com/questions/32461229/why-use-redux-over-facebook-flux)
- [Stack Overflow - 為什麼我應該要在這個範例使用 Redux？](http://stackoverflow.com/questions/35675339/why-should-i-use-redux-in-this-example)
- [Stack Overflow - 使用 Redux 取代 Flux，可能有什麼缺點？](http://stackoverflow.com/questions/32021763/what-could-be-the-downsides-of-using-redux-instead-of-flux)

<a id="general-only-react"></a>
### Redux 只能用在 React 上嗎？

Redux 可以被用作任何的 UI 層的資料儲存。最常見是用在 React 和 React Native，不過也有 Angular、Angular 2、Vue、Mithril 等等的綁定可以使用。Redux 只是簡單的提供一個訂閱的機制，它可以被用在任何其他的程式碼。也就是說它在與可以從 state 變更來推斷 UI 更新的宣告式 view 實作結合時最有用途。

<a id="general-build-tools"></a>
### 我需要特定的建構工具才能使用 Redux 嗎？

Redux 的原始碼是用 ES6 撰寫並用 Webpack 和 Babel 編譯成適合產品環境的 ES5 版本。它應該能夠用在任何的 JavaScript 建構環境。Redux 也提供一個 UMD 版本，它不需要任何建構過程就可以直接使用。[counter-vanilla](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla) 範例展示了基本的 ES5 Redux 用法，透過 `<script>` 標籤引入它。就如同相關的 Pull Request 中所說的：

> 新的 Counter 原生範例意旨在於去除 Redux 需要 Webpack、React、hot reload、sagas、action creator、constant、Babel、npm、CSS modules、decorator、fluent Latin、訂閱 Egghead、PhD，或是超過預期的 O.W.L. 水準的迷思。

> 不是的，它只需要 HTML，一些簡陋的 `<script>` tag，還有傳統的 DOM 操作。享受吧！


## Reducer

<a id="reducers-share-state"></a>
### 我要如何在兩個 reducer 之間共用 state？我一定要使用 `combineReducers` 嗎？

Redux store 建議的結構是透過 key 來把 state 物件拆分成多個「部分」或「領域」，並提供一個單獨的 reducer function 來管理每個獨立的資料部份。這是類似於標準的 Flux 模式有數個獨立的 store，而 Redux 提供了 [`combineReducers`](api/combineReducers.md) function 讓這個模式更加簡單。然而，重要的是要注意 `combineReducers` *不是*必要的 - 它是一個簡單的 function，用以針對每個 state 部分有一個 reducer function 的常見情況，並以一般的 JavaScript 物件作為資料。

許多使用者之後想要嘗試在兩個 reducer 之間共用資料，但是發現 `combineReducers` 不允許他們這麼做。這裡有幾種可以使用的方法：

* 如果 reducer 需要知道資料是來自 state 的哪個部份，store 應該需要重新被組合，讓單一的 reducer 可以處理更多的資料。
* 你可能需要撰寫一些自訂的 function 來處理一些 action。這可能需要將你的頂層 reducer function 來替代 `combineReducers`。你也可以使用一些有用的工具像是 [reduce-reducers](https://github.com/acdlite/reduce-reducers) 來執行 `combineReducers` 處理大部分的 action，同時執行更多 reducer 對於跨越不同 state 所指定的 action。
* [非同步 action creator](advanced/AsyncActions.md) 像是 `redux-thunk` 可以透過 `getState()` 存取所有的 state。Action creator 可以從 state 或是一個 action 取得額外的資料，所以每個 reducer 有更多的資訊可以更新他們本身的 state 部份。

一般情況下，記住 reducer 只是 function - 你可以根據你想要的方式來組織和區分他們，並鼓勵你將他們拆分成更小，可複用的 function （「reducer decomposition」）。你只需要確保這些 reducer 都遵守這些基本的規則： `(state, action) -> newState`，透過更新 state 來取得新的 state，而不要直接的修改 state。

#### 更多資訊

**文件**
- [API：combineReducers](api/combineReducers.md)

**討論**
- [#601 - 當一個 action 與多個 reducer 有關時，在 combineReducers 所要關注的問題](https://github.com/reactjs/redux/issues/601)
- [#1400 - 傳送頂層 state 物件到其他分支的 reducer 是一個 anti-pattern 嗎？](https://github.com/reactjs/redux/issues/1400)
- [Stack Overflow - 當使用 combine reducers，可以存取其他部分的 state 嗎？](http://stackoverflow.com/questions/34333979/accessing-other-parts-of-the-state-when-using-combined-reducers)
- [Stack Overflow - Reduce 整個 subtree 與 redux combineReducer](http://stackoverflow.com/questions/34427851/reducing-an-entire-subtree-with-redux-combinereducers)
- [在 Redux Reducer 間共用 State](https://invalidpatent.wordpress.com/2016/02/18/sharing-state-between-redux-reducers/)

<a id="reducers-use-switch"></a>
### 我一定要使用 switch 條件式來處理 action 嗎？

不是的，你可以使用任何你想要的方式在 reducer 中來應對 action。switch 條件式只是最常見的方法，但用 if 條件式、一個查詢 function 的列表，或是建立一個 function來抽象化處理也都可以。

#### 更多資訊

**文件**
- [Recipes：Reducing Boilerplate](recipes/ReducingBoilerplate.md)

**討論**
- [#883 - 消除龐大的 switch block](https://github.com/reactjs/redux/issues/883)
- [#1167 - Reducer 沒有 switch](https://github.com/reactjs/redux/issues/1167)


## 組織 State

<a id="organizing-state-only-redux-state"></a>
### 我一定要把我所有 state 放入 Redux 嗎？我應該使用 React 的 `setState()` 嗎？

這沒有「正確」的答案。有些使用者喜歡將每個單一的資料部份放入 Redux，來管理完全序列化以及在任何時候都可以控制應用程式的版本。有些人則偏好 non-critical 或 UI state，像是「目前打開的下拉選單」，內部 component 的內部 state。其實只要找到一個你覺得平衡的方式就可以了。

在 community 有各種大量不同的實作，來替代每個 component 的 state 儲存到 Redux 的 store 方法，像是 [redux-ui](https://github.com/tonyhb/redux-ui), [redux-component](https://github.com/tomchentw/redux-component)、[redux-react-local](https://github.com/threepointone/redux-react-local)，還有更多其他不同實作。

#### 更多資訊

**討論**

- [#159 - 探討使用 Redux 對於 pseudo-local component state](https://github.com/reactjs/redux/issues/159)
- [#1098 - 在可重複使用的 React component 使用 Redux](https://github.com/reactjs/redux/issues/1098)
- [#1287 - 如何在 Redux 的 store 和 React 的 state 之間做選擇？](https://github.com/reactjs/redux/issues/1287)
- [#1385 - 將你所有的 state 儲存在單一的不可變 atom 缺點是什麼？](https://github.com/reactjs/redux/issues/1385)
- [Stack Overflow - 為什麼 state 都在同一處，甚至連非全域的 state 也是？](http://stackoverflow.com/questions/35664594/redux-why-is-state-all-in-one-place-even-state-that-isnt-global)
- [Stack Overflow - 所有的 component state 都應該要保留在 Redux 的 store 嗎？](http://stackoverflow.com/questions/35328056/react-redux-should-all-component-states-be-kept-in-redux-store)

<a id="organizing-state-non-serializable"></a>
### 我可以在 store state 內放置 function、promise、或其他非序列的項目嗎？

強烈建議你在 store 只放置簡單序列化的物件、陣列、還有 primitives。在_技術上_可以新增非序列的項目到你的 store，但是這麼做會打破不可變的原則還有 store 儲存的內容。

#### 更多資訊

**討論**
- [#1248 - 在 reducer 內可以儲存一個 react component 嗎？](https://github.com/reactjs/redux/issues/1248)
- [#1279 - 在 Flux 中，有沒有將 Map Component 放在哪裡的建議？](https://github.com/reactjs/redux/issues/1279)
- [#1390 - Component 載入](https://github.com/reactjs/redux/issues/1390)
- [#1407 - 只分享一個重要的基本類別](https://github.com/reactjs/redux/issues/1407)

<a id="organizing-state-nested-data"></a>
### 我要如何在我的 state 組織巢狀和重複的資料？

一般來說，資料和 IDs、巢狀化、或是關聯的資料應該被儲存在「normalize」的方式中 - 每個物件應該只被儲存一次，透過 ID 來標記，以及其他參考的物件應該只儲存 IDs，而不是複製整個物件。它可以幫助思考對各個項目類型使用獨立「資料表」，來讓部分 store 像是一個資料庫一樣。像是 [normalizr](https://github.com/gaearon/normalizr) 和 [redux-orm](https://github.com/tommikaikkonen/redux-orm) 的 library 可以提供幫助和管理抽象的 normalize 資料。

#### 更多資訊

**文件**
- [進階：非同步 Actions](advanced/AsyncActions.md)
- [範例：Real World 範例](introduction/Examples.html#real-world)

**討論**
- [#316 - 如何建立巢狀的 reducer？](https://github.com/reactjs/redux/issues/316)
- [#815 - 處理資料結構](https://github.com/reactjs/redux/issues/815)
- [#946 - 使用拆分的 reducer 來更新相關 state 最好的方法？](https://github.com/reactjs/redux/issues/946)
- [#994 - 當更新巢狀的實體時，如何減少 boilerplate？](https://github.com/reactjs/redux/issues/994)
- [#1255 - 在 React 和 Redux 中，Normalizr 具有巢狀化物件的用法](https://github.com/reactjs/redux/issues/1255)
- [Twitter：state 形狀應該被 normalize](https://twitter.com/dan_abramov/status/715507260244496384)

## 設置 Store

<a id="store-setup-multiple-stores"></a>
### 我可以或是說我應該建立多個 store 嗎？我可以直接匯入我的 store，然後在我的 component 中使用嗎？

原來的 Flux 模式描述有多個「store」在你的應用程式，每一個 store 擁有不同的資料部份。這會引入像是 store 需要「等待」其他的 store 更新的 issue。Redux 設計上使用這個概念並將它變化，在 Redux 的 store 中，每個獨立的 Flux store 會是分離的 reducer 。

正如其他的幾個問題，在一個頁面中它是_可以_建立多個不同的 Redux store，但是預期的模式只會有一個單一的 store。然而，擁有單一的 store 可以使用 Redux DevTools，讓不變和 rehydrate 的資料更簡單，並簡化了訂閱的邏輯。

在 Redux 中使用多個 store 的一些正當理由是：

* 分析應用程式，解決頻繁更新部分 state 所造成的效能問題。
* 在大型的應用程式將 Redux 獨立做為一個 component，在這種情況下你可能想要建立每個根 component 的 store 實例。

然而，建立新的 store 應該不是你的第一個直覺，特別是如果你是從有 Flux 背景過來的人。首先，嘗試使用 reducer 做組合，如果不能解決你的問題只能使用多個 store。

同樣的，雖然你*可以*直接引入你的 store 參考實例，但這不是 Redux 推薦的模式。如果你建立一個 store 實例並從一個模組中將它匯出，它會是一個 singleton。這個意味著它可以更完善的獨立 Redux 應用程式做為一個較大的應用程式的元件，如果這不是必要的，或是啟用伺服器渲染，因為在伺服器你想要為每次的請求建立分離的 store 實例。

透過 React-Redux `connect()` function 產生的 wrapper class 確實會去尋找 `props.store` 是否存在，但是最好的方式是在你的頂部 component 使用一個 `<Provider store={store} />`，讓 React-Redux 去擔心你的 store。這種方式的 component 不需要擔心關於引入一個 store 模組，獨立一個 Redux 應用程式，或是啟用伺服器渲染變的容易許多。

#### 更多資訊

**文件**
- [API：Store](api/Store.md)

**討論**
- [#1346 - 有一個「stores」的目錄是不好的做法嗎？](https://github.com/reactjs/redux/issues/1436)
- [Stack Overflow - Redux 多個 store，有何不可？](http://stackoverflow.com/questions/33619775/redux-multiple-stores-why-not)
- [Stack Overflow - 在一個 action creatro 存取 Redux state](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator)
- [Gist：打破 Redux 規範來獨立應用程式](https://gist.github.com/gaearon/eeee2f619620ab7b55673a4ee2bf8400)

<a id="store-setup-middleware-chains"></a>
### 是不是可以有一個以上的 middleware 鏈在我的 store enhancer 中？在 middleware function 的 `next` 和 `dispatch` 的區別是什麼？

Redux middleware 的行為像是一個連結清單。每個 middleware function 可以呼叫 `next(action)` 來傳送一個 action 到下一個 middleware，呼叫 `dispatch(action)` 來重新開始處理清單，或是不做任何事以停止 action 被進一步處理。

當建立 store 時，一連串的 middleware 透過參數來定義並被傳送到 `applyMiddleware` function。定義多個 middleware 鏈無法正常的執行，因為他們明顯有不同參考的 `dispatch`，而且不同的鏈可以有效的被分開。

#### 更多資訊

**文件**
- [進階：Middleware](advanced/Middleware.md)
- [API：applyMiddleware](api/applyMiddleware.md)

**討論**
- [#1051 - 目前 applyMiddleware 和 createStore compse 的缺點](https://github.com/reactjs/redux/issues/1051)
- [理解 Redux Middleware](https://medium.com/@meagle/understanding-87566abcfb7a)
- [探索 Redux Middleware](http://blog.krawaller.se/posts/exploring-redux-middleware/)

<a id="store-setup-subscriptions"></a>
### 我要如何只訂閱部份的 state？我可以取得被 dispatch 的 action 當做訂閱的一部分嗎？

Redux 提供了 `store.subscribe` 方法來通知 listeners store 已經更新。Listener callbacks 不會將目前取得的 state 當作參數 - 他只是指出_那些_ state 已經改變。訂閱者可以呼叫 `getState()` 來取得目前的值。

這個 API 被作為一個底層的 primitive 沒有任何的依賴或是複雜性，而且可以用在建立頂層的訂閱邏輯。像是 React-Redux 的 UI binding 可以建立一個訂閱給每個被連結的 component。它也可以撰寫 function，而且可以聰明地比較舊的和新的 state，如果某些 state 改變了也可以執行附帶的邏輯。範例包含了 [redux-watch](https://github.com/jprichardson/redux-watch) 和 [redux-subscribe](https://github.com/ashaffer/redux-subscribe)，提供不同的方法來指定訂閱和處理變化。

為了排除特殊的情況和簡化實作的附加功能，像是 `Redux DevTools`，新的 state 不會傳送到 listeners。另外，訂閱者的目的是反應 state 本身，而不是 action，如果需要處理 action 可以使用 Middleware。

#### 更多資訊

**文件**
- [基礎：Store](basics/Store.md)
- [API：Store](api/Store.md)

**討論**
- [#303 - 訂閱 API 和 state 作為參數](https://github.com/reactjs/redux/issues/303)
- [#580 - 在 store.subscribe 是可以取得 action 和 state 嗎？](https://github.com/reactjs/redux/issues/580)
- [#922 - 建議：加入訂閱到你的 middleware API](https://github.com/reactjs/redux/issues/922)
- [#1057 - 訂閱 listener 可以取得 action 的參數嗎？](https://github.com/reactjs/redux/issues/1057)
- [#1300 - Redux 很棒但缺少主要功能](https://github.com/reactjs/redux/issues/1300)

## Action

<a id="actions-string-constants"></a>
### 為什麼 `type` 要是一個字串，或是至少可以 serialize 的？為什麼我的 action type 應該要是常數？

與 state 相同，讓 action 序列化將可使用一些 Redux 定義的功能，像是 time travel debugging，以及 recording 和 replaying actions。使用一些像是 Symbol 的 「type」的值可能會破壞這些。字串序列化可以簡單的描述本身，所以是更好的選擇。請注意，如果 action 是要提供給 middleware 使用，在 action 內可以使用 Symbol、Promise，或是其他非序列化的數值 - 在實際到達 store 和傳送給 reducer 時，action 只需要可以被序列化。

由於效能原因，我們不能依靠執行去序列化的 action，所以 Redux 只確認每個 action 是普通的物件，並定義 `type`。剩下的就看你自己了，但你可能會發現保持可序列化可以幫助 debug 和重現問題。

封裝和集中常用的程式碼部份是程式設計中的一個關鍵概念。雖然每個部份可能都要手動建立 action 物件，還要手動撰寫每個 `type` 的值，定義可複用的常數讓你維護程式碼更容易。如果你將常數放在一個單獨檔案，你可以[針對 `import` 宣告做確認](https://www.npmjs.com/package/eslint-plugin-import)，因為你可能不小心使用了錯誤的字串。

#### 更多資訊

**文件**
- [Reducing Boilerplate](http://rackt.github.io/redux/docs/recipes/ReducingBoilerplate.html#actions)

**討論**
- [#384 - 推薦 Action 常數使用過去時態來命名](https://github.com/reactjs/redux/issues/384)
- [#628 - 簡單的 action create 和更少的 boilerplate 的解決方案](https://github.com/reactjs/redux/issues/628)
- [#1024 - 建議：宣告 reducer](https://github.com/reactjs/redux/issues/1024)
- [#1167 - Reducer 沒有 switch](https://github.com/reactjs/redux/issues/1167)
- [Stack Overflow - 在 Redux 為什麼需要將「Action」當作資料？](http://stackoverflow.com/q/34759047/62937)
- [Stack Overflow - 在 Redux 常數的點是什麼？](http://stackoverflow.com/q/34965856/62937)

<a id="actions-reducer-mappings"></a>
### reducer 和 action 之間一定是一對一互相對應嗎？

不是的。實際上，建議的模式是各個 reducer function 負責各部分的 state 進行更新 - 「reducer composition」模式。一個指定的 action 可以處理所有或是部份 state，也可以沒有。一個指定的 action 可以處理所有或是部份 state，或是完全沒有。有些使用者選擇將 action 和 reducer 更緊密的 bind 在一起，像是「duck」檔案結構， 但預設上不一定要一對一互相對應。

#### 更多資訊

**文件**
- [基礎：Reducer](basics/Reducers.md)

**討論**
- [Twitter - Redux 最常見的誤解](https://twitter.com/dan_abramov/status/682923564006248448)
- [#1167 - Reducer 沒有 switch](https://github.com/reactjs/redux/issues/1167)
- [Reduxible #8 - Reducer 和 action creator 不是一對一的對應](https://github.com/reduxible/reduxible/issues/8)
- [Stack Overflow - 我可以 dispatch 多個 action 而不需要 Redux Thunk middleware 嗎？](http://stackoverflow.com/questions/35493352/can-i-dispatch-multiple-actions-without-redux-thunk-middleware/35642783)

<a id="actions-side-effects"></a>
### 我如何表達像是 AJAX 呼叫之類的「side effect」？為什麼我們需要像是「action creator」、「thunk」、以及「middleware」來執行非同步的行為？

這是一個冗長以及複雜的問題，要如何將各式各樣的程式碼組織以及應該使用什麼樣的方法。

任何有意義的 web 應用程式都會需要執行複雜的邏輯，通常包括像是非同步的 AJAX 請求。程式碼不再是單純的輸入 function，以及和外部世界之間相互作用被稱為[「side effects」](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29)。

Redux 的靈感來自於 functional programming，它是可以直接使用的，不會有任何的 side effects 執行。尤其是 reducer function，它 **必須** 是 `(state, action) -> newState` 的 pure function。然而，Redux 的 middleware 讓它可以攔截被 dispatch 的 action 並附加額外的複雜行為，包含 side effects。

一般來說，Redux 建議具有 side effect 的程式碼應該是 action creation 處理的一部分。雖然邏輯上_可以_在 UI component 內部被執行，更有意義的方式是將可複用的 function 抽出，相同的邏輯可以在其他地方被呼叫使用 - 換句話說，是一個 action creator function。

最簡單和最常見的方式是增加 [redux-thunk](https://github.com/gaearon/redux-thunk) middleware，讓你可以撰寫更複雜或是非同步邏輯的 action creator。一個被廣泛使用的方法是 [redux-saga](https://github.com/yelouafi/redux-saga)，它使用 generator 讓你可以撰寫更多像是同步的程式碼，可以像是「背景執行序」或「daemons」在你的 Redux 應用程式。另一個方法是 [redux-loop](https://github.com/raisemarketplace/redux-loop)，允許你的 reducer 來宣告 side effects 在回應時的 state 改變，並讓他們分開執行反轉的處理。此外，有_許多_其他由 community 所開發的 library 以及 ideas，每個 libray 都有自己管理 side effects 的方法。


#### 更多資訊
**文件**
- [進階：Async Actions](advanced/AsyncActions.md)
- [進階：Async Flow](advanced/AsyncFlow.md)
- [進階：Middleware](advanced/Middleware.md)

**討論**
- [#291 - 嘗試把呼叫 API 放在正確的地方](https://github.com/reactjs/redux/issues/291)
- [#455 - 塑造 side effects](https://github.com/reactjs/redux/issues/455)
- [#533 - 簡單介紹非同步 action creator](https://github.com/reactjs/redux/issues/533)
- [#569 - 建議：API 對於顯式的 side effects](https://github.com/reactjs/redux/pull/569)
- [#1139 - 基於 generators 和 sagas 替代 side effect model](https://github.com/reactjs/redux/issues/1139)
- [Stack Overflow - 在 Redux 為什麼我們需要 middleware 來處理非同步的流程？](http://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux)
- [Stack Overflow - 如何 dispatch 一個 timeout 的 Redux action？](http://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559)
- [Stack Overflow - 在 Redux 哪邊應該是我放置同步 side effects 連結到 action 的地方？](http://stackoverflow.com/questions/32982237/where-should-i-put-synchronous-side-effects-linked-to-actions-in-redux/33036344)
- [Stack Overflow - 在 Redux 如何處理 side-effects？](http://stackoverflow.com/questions/32925837/how-to-handle-complex-side-effects-in-redux/33036594)
- [Stack Overflow - 如何在單元測試非同步 Redux action 來 mock ajax 的 回應？](http://stackoverflow.com/questions/33011729/how-to-unit-test-async-redux-actions-to-mock-ajax-response/33053465)
- [Stack Overflow - 如何在 Redux 回應 state 的改變來觸發呼叫 AJAX？](http://stackoverflow.com/questions/35262692/how-to-fire-ajax-calls-in-response-to-the-state-changes-with-redux/35675447)
- [Reddit - 說明執行非同步 API 呼叫與 Redux-Promise Middleware。](https://www.reddit.com/r/reactjs/comments/469iyc/help_performing_async_api_calls_with_reduxpromise/)
- [Twitter - 在 sagas、loops、以及其他方法之間的比較](https://twitter.com/dan_abramov/status/689639582120415232)
- [Redux Side-Effects 和你](https://medium.com/@fward/redux-side-effects-and-you-66f2e0842fc3)
- [在 Redux 的 pure functionality 和 side effects](http://blog.hivejs.org/building-the-ui-2/)

<a id="actions-multiple-actions"></a>
### 我應該從 action creator 連續 dispatch 多個 action 嗎？

沒有具體的規定你該如何建構你的 action。使用一個非同步 middleware 像是 redux-thunk，像是 dispatch 連續多個但是不相關的 action，dispatch action 來表示 AJAX 請求、dispatch action 基於 state 的狀態、 甚至是 dispatch 一個 action，然後立即檢查更新後的狀態。

一般來說，這些 action 是相關但是獨立的，或者實際上可能是相同的。根據你的情況使用有意義的方式，並基於可讀性和 debug。

如果你擔心效能問題的話，盡量避免在一個地方同步連續 dispatch。如果你使用 React，注意到你可以在 `ReactDOM.unstable_batchedUpdates()`  wrap 多個同步的 dispatch 提高效能，但是這個 API 是實驗性的，可能會在任何 React  被移除，所以不要過度依賴。[redux-batched-actions](https://github.com/tshelburne/redux-batched-actions) 在 reducer 就好像是一個「unpack」，讓你 dispatch 多個 action，[redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) 讓你可以訂閱呼叫多個 dispatch。

#### 更多資訊

**討論**
- [#597 - 從一個 event handler 有效的 dispatch 多個 action？](https://github.com/reactjs/redux/issues/597)
- [#959 - 多個 action 在一個 dispatch？](https://github.com/reactjs/redux/issues/959)
- [Stack Overflow - 我應該使用一個或多個 action type 來表示非同步的 action 嗎？](http://stackoverflow.com/questions/33637740/should-i-use-one-or-several-action-types-to-represent-this-async-action/33816695)
- [Stack Overflow - 在 Redux 執行事件和 action 是一對一的關係嗎？](http://stackoverflow.com/questions/35406707/do-events-and-actions-have-a-11-relationship-in-redux/35410524)
- [Stack Overflow - 應該由 reducer action 來處理相關 action 或者由 action creator 本身來產生 action 嗎？](http://stackoverflow.com/questions/33220776/should-actions-like-showing-hiding-loading-screens-be-handled-by-reducers-to-rel/33226443#33226443)


## 程式碼結構

<a id="structure-file-structure"></a>
### 我的檔案結構應該長什麼樣子？在我的專案中，我應該如何組織我的 action creator 和 reducer？我的 selector 應該放哪？

因為 Redux 只是一個資料 store library，它沒有直接的方法來如何建構你的專案。然而，有少數幾個是 Redux 開發者最常傾向使用的模式：

- Rails 風格：「actions」、「constants」、「reducers」、「containers」、「components」獨立的資料夾。
- Domain 風格：每個功能或 domain 單獨的資料夾，可能有每個檔案類型的子資料夾。
- 「Ducks」：類似於 domain 風格，但是明確的把 action 和 reducer 綁在一起，通常會將他們定義在同個檔案中。

一般認為，selector 應該和 reducer 一起被定義然後匯出，接著在其他地方重新使用（像是在 `mapStateToProps` functions 或者是在非同步 action creators）來隱藏實際的 store 形狀。

#### 更多資訊

**討論**
- [#839 - 強調與 reducer 一起定義 selector ](https://github.com/reactjs/redux/issues/839)
- [#943 - Reducer 查詢](https://github.com/reactjs/redux/issues/943)
- [React-Boilerplate #27 - 應用程式結構](https://github.com/mxstbr/react-boilerplate/issues/27)
- [Stack Overflow - 如何建構 Redux components 和 containers？](http://stackoverflow.com/questions/32634320/how-to-structure-redux-components-containers/32921576)
- [Redux 最佳實踐](https://medium.com/lexical-labs-engineering/redux-best-practices-64d59775802e)
- [對於結構化 (Redux) 應用程式的規則](http://jaysoo.ca/2016/02/28/organizing-redux-application/)
- [A Better File Structure for React/Redux Applications](http://marmelab.com/blog/2015/12/17/react-directory-structure.html)
- [Organizing Large React Applications](http://engineering.kapost.com/2016/01/organizing-large-react-applications/)
- [Four Strategies for Organizing Code](https://medium.com/@msandin/strategies-for-organizing-code-2c9d690b6f33)

<a id="structure-business-logic"></a>
### 我應該如何將我的 reducer 和 action creator 之間的邏輯拆開？我的「商業邏輯」應該放在哪裡？

邏輯放置的部份沒有一個明確的答案應該在 reducer 或是 action creator。有些開發者偏好「fat」action creator、「thin」reducer，action 可以簡單的將資料 merge 到相對應的 state，其他人盡可能嘗試讓 action 保持簡單，在 action creator 盡量避免使用到 `getState`。

這個討論總結了一個不錯的二分法 ︰

> 現在的問題是要將什麼放在 action creator、什麼要在 reducer 中，以及 fat 和 thin action 物件之間的選擇。如果你把所有的邏輯放在 action creator，你最終的 fat action 物件基本上是宣告 state 更新的結果。Reducer 是 pure、dumb、新增、移除、更新的這些功能。他們會更容易 compose。但不代表你大部分的商業邏輯都會在那裡。
> 你如果在 reducer 上放了許多邏輯，你就可以有 thin action 物件，大部分的資料邏輯都在一個地方，但是你的 reducer 會很困難 compose，因為你可能需要來自其他 branch 的資訊。你最後可能有一個很大的 reducer 或是 reducer 帶有其他額外的參數來自上層的 state。

在這兩個極端之間找到平衡，你就可以掌握 Redux。

#### 更多資訊

**討論**
- [#1165 - 如何放置商業邏輯和驗證？](https://github.com/reactjs/redux/issues/1165)
- [#1171 - 關於 action-creators、reducers、和 selectors 最佳實踐與建議](https://github.com/reactjs/redux/issues/1171 )
- [Stack Overflow - 在一個 action creator 存取 Redux state？](http://stackoverflow.com/questions/35667249/accessing-redux-state-in-an-action-creator/35674575)


## 效能

<a id="performance-scaling"></a>
### 依據效能和架構方面來說，Redux 能「scale」的多好？

依然沒有一個明確的答案，大多數這個時候不應該關注這兩種情況。

一般 Redux 所作的工作分為幾個領域：處理在 middleware 的 action 和 reducer（包含複製不可變的物件來進行更新），在 action 被 dispatch 後通知訂閱者，以及基於 state 的改變來更新 UI component。雖然這些很有_可能_在充滿複雜的情況下變成性能上的問題，但 Redux 的實作本身沒有什麼什麼緩慢和低效的問題。事實上，React-Redux 特別做了大量的優化，減少了不必要的重新 render。

在結構當中，Redux 在各個專案和團隊中都工作的非常順利，數百家公司和上千位開發者目前都在使用 Redux，在 NPM 達到一個月幾十萬次的安裝。一位開發者的報告：

> 在規模上，我們有 ~500 action types、~400 reducer cases、~150 components、5 middlewares、~200 actions、 ~2300 tests。

#### 更多資訊

**討論**
- [#310 - 誰在使用 Redux？](https://github.com/reactjs/redux/issues/310)
- [Reddit - 保持初始 state 最好的地方是什麼？](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)
- [Reddit - 設計一個單頁應用程式的 Redux state 說明](https://www.reddit.com/r/reactjs/comments/48k852/help_designing_redux_state_for_a_single_page/)
- [Reddit - Redux 的效能 issue 和大量 state 物件？](https://www.reddit.com/r/reactjs/comments/41wdqn/redux_performance_issues_with_a_large_state_object/)
- [Reddit: React/Redux for Ultra Large Scale apps](https://www.reddit.com/r/javascript/comments/49box8/reactredux_for_ultra_large_scale_apps/)
- [Twitter - Redux scaling](https://twitter.com/NickPresta/status/684058236828266496)

<a id="performance-all-reducers"></a>
### 針對每個 action 呼叫「我所有的 reducer」不會很慢嗎？

重要的是，注意到一個 Redux store 只有一個單一的 reducer function。store 傳送目前的 state 和 dispatch action 到一個 reducer function，並讓 reducer 做適當的處理。

很明顯的，將單一的 function 擴充並嘗試處理每個 action 不是這麼容易，簡單來說在於 function 的大小和可讀性，所以要將實際的工作分離成獨立的 function 讓頂層的 reducer 可以呼叫。特別是，共同建議的模式有一個獨立的 reducer function 根據指定的 key 負責管理部份的 state 的更新，透過使用提供的 `combineReducers` 工具可以讓程式變得更簡單。但還是強烈建議將你的 store 扁平化、序列化。最後，你可以根據你需要的方式來組織 reducer 邏輯。

然而，即便剛好你有許多不同獨立的 reducer，甚至有很深的巢狀 state，reducer 的速度不太可能是一個問題。JavaScript Engine 可以在每秒呼叫並執行大量的 function，你大部份的 reducer 可能都使用 switch 語句，預設上大部分 action 的回應都會回傳存在的 state。

如果你很擔心 reducer 實際上的效能，你可以是用像是 [redux-ignore](https://github.com/omnidan/redux-ignore) 或 [reduxr-scoped-reducer](https://github.com/chrisdavies/reduxr-scoped-reducer) 的工具來確保某些 reducer 只監聽指定的 action。你還可以使用 [redux-log-slow-reducers](https://github.com/michaelcontento/redux-log-slow-reducers) 來做一些效能的基準測試。

#### 更多資訊

**討論**
- [#912 - 建議：action filter 工具](https://github.com/reactjs/redux/issues/912)
- [#1303 - Redux 效能和頻繁的更新大型的 store](https://github.com/reactjs/redux/issues/1303)
- [Stack Overflow - 在 Redux app state 要有 reducer 的名稱](http://stackoverflow.com/questions/35667775/state-in-redux-react-app-has-a-property-with-the-name-of-the-reducer/35674297)
- [Stack Overflow - Redux 如何處理深層巢狀的 model？](http://stackoverflow.com/questions/34494866/how-does-redux-deals-with-deeply-nested-models/34495397)

<a id="performance-clone-state"></a>
### 我需要在我的 reducer 深層複製我的 state 嗎？複製 state 不會導致變慢嗎？

Immutable 的更新 state 一般的意思是說淺層複製，而不是深層複製。淺層複製比深層複製來的快，因為較少的物件和欄位被複製，以及可以有效的歸納變動的部份。

然而，你需要_將_每個受影響的巢狀層面，建立一個複製和更新的物件。雖然這應該不是特別昂貴，這是一個很好的理由為什麼你應該保持 state 的序列化以及淺層複製。

> Redux 常見的誤解：你需要深層 clone state。實際上：如果內部的 state 沒有改變，保持相同的參考！

#### 更多資訊

**討論**
- [#454 - 在 reducer 處理大型的 state](https://github.com/reactjs/redux/issues/454)
- [#758 - 為什麼 state 不能被 mutate？](https://github.com/reactjs/redux/issues/758)
- [#994 - 當更新巢狀實體時，如何減少 boilerplate？](https://github.com/reactjs/redux/issues/994)
- [Twitter - 常見的誤解 - 深層 clone](https://twitter.com/dan_abramov/status/688087202312491008)
- [在 JavaScript 中 clone 物件](http://www.zsoltnagy.eu/cloning-objects-in-javascript/)

<a id="performance-update-events"></a>
### 我如何減少 store 更新事件的次數？

Redux 在每次成功 dispatch action 會通知訂閱者（意思是，action 已經去改變 store，並透過 reducer 來處理）。在有些情況下，它可能對於減少呼叫多次訂閱者是有幫助的，特別是如果一個 action creator 連續 dispatch 多個不同的 action。在 community 有一些 addons，在大量的 action 被 dispatch 後，提供透過批次的方式來通知訂閱者，像是 [redux-batched-updates](https://github.com/acdlite/redux-batched-updates)、[redux-batched-subscribe](https://github.com/tappleby/redux-batched-subscribe) 或 [redux-batched-actions](https://github.com/tshelburne/redux-batched-actions)。

#### 更多資訊

**討論**
- [#125 - 避免 cascade render 的策略](https://github.com/reactjs/redux/issues/125)
- [#542 - 方法：batching actions](https://github.com/reactjs/redux/issues/542)
- [#911 - Batching actions](https://github.com/reactjs/redux/issues/911)
- [React-Redux #263 - 當 dispatch 數百個 action，造成的龐大的效能 issue](https://github.com/reactjs/react-redux/issues/263)

<a id="performance-state-memory"></a>
### 「一個 state tree」會造成記憶體上的問題嗎？dispatch 許多 action 會佔用記憶體嗎？

首先，在原始記憶體的使用方面，Redux 和其他的 JavaScript libray 沒有什麼不同。唯一的區別是所有的物件參考都被巢狀化成一個 tree，而不是儲存在各個獨立的 model 實例，像是：Backbone。第二，一個典型的 Redux app 記憶體使用 _稍微_ 會比 Backbone app 來的少，因為 Redux 鼓勵你使用純 JavaScript 物件和陣列，而不是建立 Model 和 Collection 的實例。最後，Redux 在每個時間點只有單一 state tree 的參考。根據標準，物件如果不再被 tree 參考將會被垃圾回收。

Redux 不會 store action 本身的歷史記錄。然而，Redux DevTools 可以 store action 所以他們可以被 replay，但是一般只會在開發期間使用，不會使用在上線環境。

#### 更多資訊

**文件**
- [文件：非同步的 Action](advanced/AsyncActions.md])

**討論**
- [Stack Overflow - 在 Redux 有沒有任何方法來「commit」state 來釋放記憶體？](http://stackoverflow.com/questions/35627553/is-there-any-way-to-commit-the-state-in-redux-to-free-memory/35634004)
- [Reddit -保持初始 state 的地方是什麼？](https://www.reddit.com/r/reactjs/comments/47m9h5/whats_the_best_place_to_keep_the_initial_state/)

## React Redux

<a id="react-not-rerendering"></a>
### 為什麼我的 component 不會重新 render，或者是執行 mapStateToProps？

為什麼 component 在 action 被 dispatch 後不會重新 render，到目前為止最常見的原因是因為你無意間直接 mutate 或修改你的 state。Redux 期望你的 reducer 更新 state 是「immutably」的，這實際上的意思就是你需要複製資料，並將改變的部分應用到副本。如果你從 reducer 回傳相同的物件，Redux 會假設一切都沒有改變，即使你對內容進行修改。同樣的，React-Redux 透過 `shouldComponentUpdate` 在淺層平等參考來檢查傳入的 props 去嘗試改善效能，如果所有的參考都是相同的，回傳 false 來跳過實際更新你的原始 component。

很重要的是你要記得，每當你更新巢狀的值，你必須在你的 state tree 中回傳任何以上的新副本。如果你有 `state.a.b.c.d`，然你想要更新 `d`，你應該需要回傳 `c`、`b`、`a` 以及 `state` 的新副本。這個 [state tree mutation 關係圖](http://arqex.com/wp-content/uploads/2015/02/trees.png) 表示了如何改變在 tree 深層的 state，需要一路的往上改變。

注意到「immutably 的更新資料」意思_不是_你必須使用 Immutable.js library，儘管這當然是一種選擇。你可以透過純 JavaScript 物件和陣列使用不同的方式來達到 immutable 的更新資料：

- 使用 function 像是 Object.assign() / \_.extend() 來複製物件，陣列的話使用像是 slice() 和 concat() 的 function。
- 在 ES6 陣列的展開運算符，還有類似物件的展開運算符，已經提出建議了但是還沒有通過。
- 有用的 library 可以將 immutable 的更新邏輯包裝成簡單的 function。

#### 更多資訊

**文件**
- [Troubleshooting](Troubleshooting.md)
- [React-Redux：Troubleshooting](https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md)
- [Recipes：使用物件展開運算符](recipes/UsingObjectSpreadOperator.md)

**討論**
- [#1262 - Immutable 資料 + 不佳的效能](https://github.com/reactjs/redux/issues/1262)
- [React-Redux #235 - Predicate function 來更新 component](https://github.com/reactjs/react-redux/issues/235)
- [React-Redux #291 - action 每次被 dispatch 時，mapStateToProps 應該會被呼叫嗎？](https://github.com/reactjs/react-redux/issues/291)
- [Stack Overflow -  在 Redux 有乾淨或簡短的方式來更新巢狀的 state 嗎？](http://stackoverflow.com/questions/35592078/cleaner-shorter-way-to-update-nested-state-in-redux)
- [Gist - state mutations](https://gist.github.com/amcdnl/7d93c0c67a9a44fe5761#gistcomment-1706579)
- [React 使用 Immutability 的利與弊](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/)

<a id="react-rendering-too-often"></a>
### 為什麼我的 component 會重新 render 過於頻繁？

React-Redux 實作了許多優化來確保你實際的 component 只有當必要時才重新 render。傳送到 `connect` 的 `mapStateToProps` 和 `mapDispatchToProps` 參數，在產生 combine props 物件時，其中之一會進行淺平等的檢查。不幸的是，淺平等沒辦法幫助在每次 `mapStateToProps` 被呼叫時，新的陣列和物件實例被建立的情況。一個典型的例子可能是 map 所有陣列裡的 IDs 以及回傳對應到的參考物件，像是：

```js
const mapStateToProps = (state) => {
  return {
    objects: state.objectIds.map(id => state.objects[id])
  }
}
```

儘管每次陣列可能有完全相同的參考物件，陣列本身是一個不同的參考，所以 shallow equality 會確認失敗，React-Redux 會重新 render 被包覆的 component。

透過儲存物件的陣列，使用 reducer 進入到 state，可以解決額外的重新 render，快取和 memoize 使用 Reselect library 來 map 陣列，或是在你的 component 實作 `shouldComponentUpdate` ，使用像是 `_.isEqual` 的 function 做更深入的 props 對照。

對於非 connect 的 component，你可能會想要檢查哪些 props 被傳送了。一個常見的 issue 是有一個父 component 重新 bind 一個 callback 在 render function 內，像是 `<Child onClick={this.handleClick.bind(this)} />`。每次父 component 重新 render，會建立一個新的參考 function。在你的父 component 的 constructor 中，一般好的作法只能 bind callback 一次。

#### 更多資訊

**討論**
- [Stack Overflow - React-Redux app 可以 scale 像 Backbone 嗎？](http://stackoverflow.com/questions/34782249/can-a-react-redux-app-really-scale-as-well-as-say-backbone-even-with-reselect)
- [React.js pure render 效能 anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)
- [A Deep Dive into React Perf Debugging](http://benchling.engineering/deep-dive-react-perf-debugging/)

<a id="react-mapstate-speed"></a>
### 我如何讓我的 mapStateToProps 變快？

當你的 `mapStateToProps` function 被呼叫時，React-Redux 盡量減少工作的次數，這是一個好的想法來確保你的 `mapStateToProps` 執行快速，而且也減少大量的工作時間。普遍推薦的方法是使用 [Reselect](https://github.com/reactjs/reselect) library 建立 memoize 的「selector」function。這些 selector 可以一起被 combine 和 compose，在傳遞過程之後的的 selector，只有當輸入有變化時才會執行。這意思說你可以建立像是過濾和排序的 selector，如果需要的話，確保只發生在真實的工作上。

#### 更多資訊

**文件**
- [Recipes：計算派送的資料](recipes/ComputingDerivedData.md)

**討論**
- [#815 - 處理資料結構](https://github.com/reactjs/redux/issues/815)
- [Reselect #47 - Memoize 分層 Selectors](https://github.com/reactjs/reselect/issues/47)

<a id="react-props-dispatch"></a>
### 為什麼我在 connect 的 component 沒有 `this.props.dispatch` 可以使用？

`connect` function 有兩個主要參數，兩者是可選的。首先，`mapStateToProps`，它是一個 funcion，當 store 發生改變時，你提供一個方法從 store 將 state 拉回，然後將這些值當作 props 傳送到你的 component。第二，`mapDispatchToProps`，它是一個 function，讓你提供 store 的 dispatch function 來使用，通常透過建立 action creator 預先綁定的版本，當他們被呼叫時可以自動 dispatch 這些 action。

如果當呼叫 `connect` 時，你不想提供你的 `mapDispatchToProps` function，React-Redux 會提供一個預設的版本，簡單的回傳 `dispatch` 當作 props。這意思說，假設你_提供_自己的 function，`dispatch` 就_不會_自動的提供了。如果仍然想要可以當作 props 來使用，在你的 `mapDispatchToProps`  實作你需要顯式回傳它。

#### 更多資訊

**文件**
- [React-Redux API：connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)

**討論**
- [React-Redux #89 - 可以將多個 actionCreators 包裝到一個有名稱的 props 嗎？](https://github.com/reactjs/react-redux/issues/89)
- [React-Redux #145 - 始終考慮如何傳送 dispatch，而不管 mapDispatchToProps 是怎麼做](https://github.com/reactjs/react-redux/issues/145)
- [React-Redux #255 - 如果使用 mapDispatchToProps this.props.dispatch 是 undefined](https://github.com/reactjs/react-redux/issues/255)
- [Stack Overflow - 在 Redux 如何使用 connect 簡單從 this.props 取得 dispatch？](http://stackoverflow.com/questions/34458261/how-to-get-simple-dispatch-from-this-props-using-connect-w-redux/34458710)

<a id="react-multiple-components"></a>
### 我應該只 connect 我的頂層 component，或者我可以在我的 tree 中 connect 更多的 component？

早期的 Redux 文件建議你應該只能有幾個在你頂層附近 component tree 可以被 connect。然而，時間和經驗可以表示，一般來說，幾個 component 知道太多關於其他子 component 的資料要求，並強迫它們往下傳遞大量令人困惑的 props。

目前建議的最好的做法是將你的 component 分類成「presentational」或「container」，無論怎麼樣，確實的 connect container component：

> 強調「一個 conatiner component 在頂層」在 Redux 範例中是一個錯誤，不要認為這是一句名言。嘗試將你的 presentation component 保持獨立。透過 connect 他們來建立 container component 會更方便。每當你感覺你在父 component 複製程式碼，來提供資料給相同的子 component，就是你抽出 container 的時候。一般你只要感覺到父 component 知道太多關於子 component 的「個人」的資料或 action，就是你抽出 container 的時候。

一般情況下，嘗試理解資料流和你的 component 的責任，並在這之間找到一個平衡。

#### 更多資訊

**文件**
- [基礎：與 React 的使用方法](basics/UsageWithReact.md)

**討論**
- [Presentational 和 Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [Twitter - 強調「一個 container」是一個錯誤](https://twitter.com/dan_abramov/status/668585589609005056)
- [#419 - connect 的建議使用方法](https://github.com/reactjs/redux/issues/419)
- [#756 - container vs component？](https://github.com/reactjs/redux/issues/756)
- [#1176 - Redux+React 與唯一 stateless components](https://github.com/reactjs/redux/issues/1176)
- [Stack Overflow - dumb component 可以使用 Redux container 嗎？](http://stackoverflow.com/questions/34992247/can-a-dumb-component-use-render-redux-container-component)

## 雜項

<a id="miscellaneous-real-projects"></a>
### 有沒有任何更大的、「實際的」Redux 專案？

Redux 的「範例」資料夾有很多各種不同複雜程度的 sample 專案，包含一個「real-world」範例。雖然許多公司都正在使用 Redux，但他們的應用程式都是版權而不可使用的。A large number of Redux-在 Github 上可以找到大量相關的 Redux 專案，像是 [Stack Overflowund-Redux](https://github.com/andrewngu/sound-redux)。

#### 更多資訊

**文件**
- [介紹：範例](introduction/Examples.md)

**討論**
- [Reddit - 大型 open source react/redux 專案？](https://www.reddit.com/r/reactjs/comments/496db2/large_open_source_reactredux_projects/)
- [HN - 有任何使用 Redux 建立的大型 web 應用程式嗎？](https://news.ycombinator.com/item?id=10710240)

<a id="miscellaneous-authentication"></a>
### 我該如何在 Redux 中實作認證？

認證在任何真實的應用程式都是必要的。有關認證的時候你必須記得，這對於你應該如何組織你的應用程式一點都沒有影響，你應該以同樣的方式來實作認證。它非常簡單：

1. 為成功登入和登入失敗或是其他的動作，建立 action constant

2. 建立 actionCreators 帶有一個 type、憑證、一個 flag 來表示認證是 true 或 false、一個 token、或一個做為資料的錯誤訊息

3. 建立一個非同步的 action creator 和 redux-thunk middleware 或任何你覺得適合觸發網路請求的 middleware 到一個 api，如果憑證有效，回傳一個 token 並可以儲存到 local storage 或一個 response 給使用者，如果要處理失敗的話，透過步驟二中所建立合適的 actionCreators 來處理。

4. 建立一個 reducer，在每個可能認證的情況（登入成功、登入失敗）來回傳 next state。

#### 更多資訊

**討論**
- [Authentication with JWT by Auth0](https://auth0.com/blog/2016/01/04/secure-your-react-and-redux-app-with-jwt-authentication/)
- [在 Redux 處理認證的 tips](https://medium.com/@MattiaManzati/tips-to-handle-authentication-in-redux-2-introducing-redux-saga-130d6872fbe7)
- [react-redux-jwt-auth-example](https://github.com/joshgeller/react-redux-jwt-auth-example)
- [redux-auth](https://github.com/lynndylanhurley/redux-auth)
