# 先前技術

Redux 有許多以前的人留下來的東西。它相似於一些模式與技術，但也在某些重要的地方與它們不同。我們將會在下面探討一些相似與差異之處。

### Flux

Redux 可以被想成是一種 [Flux](https://facebook.github.io/flux/) 實作嗎？
[是](https://twitter.com/fisherwebdev/status/616278911886884864)，同時也[不是](https://twitter.com/andrestaltz/status/616270755605708800)。

(不用擔心，[Flux 的作者們](https://twitter.com/jingc/status/616608251463909376)[也同意它](https://twitter.com/fisherwebdev/status/616286955693682688)，如果這就是你想知道的。)

Redux 受到幾個 Flux 的重要特質所影響。就像 Flux 一樣，Redux 也規定你要把 model 的更新邏輯集中在應用程式的特定一層 (在 Flux 裡是「stores」，在 Redux 裡是「reducers」)。它們都告訴你要把每一個變動描述成一個被叫做「action」的一般物件，而不是讓應用程式的程式碼直接變動資料。

跟 Flux 不一樣的是，**Redux 沒有 Dispatcher 的概念**。這是因為它依靠 pure function 而不是 event emitter，而且組合 pure functions 很簡單而不需要額外的實體來管理它們。根據你如何看待 Flux，你可以把這個看作是一個偏差也可以看作是一個實作細節。Flux 時常被[描述為 `(state, action) => state`](https://speakerdeck.com/jmorrell/jsconf-uy-flux-those-who-forget-the-past-dot-dot-dot-1)。從這點來看，Redux 是真的是 Flux 架構，不過由於 pure functions 讓它更簡單。

另一個跟 Flux 重要的不同是 **Redux 假設你永遠不會變動你的舊資料**。你可以使用一般物件或陣列來當作 state 都沒關係，但是強烈不鼓勵在 reducers 裡面變動它們。你應該總是回傳一個新的物件，而這很容易藉由[被提議成 ES7 的 object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread) 達成並且它已經被 [Babel](http://babeljs.io) 實作，或是用一個像是 [Immutable](https://facebook.github.io/immutable-js) 的 library。

雖然技術上*可以*[寫 impure 的 reducers](https://github.com/rackt/redux/issues/328#issuecomment-125035516) 來針對效能上的極端案例變動資料，但我們強烈不鼓勵你這樣做。一些開發用的功能像是 time travel、記錄/重播、或是 hot reloading 將會壞掉。此外在大部份實際的應用程式中 immutability 看來並不會被提出效能問題，因為就如 [Om](https://github.com/omcljs/om) 所展示的，即使你輸在物件分配，你仍然藉由避免昂貴的重新 render 和重新計算取勝，因為多虧 reducer purity 你知道究竟什麼東西改變了。

### Elm

[Elm](http://elm-lang.org/) 是一個 [Evan Czaplicki](https://twitter.com/czaplic) 創造並受到 Haskell 影響的 functional programming language。它強制[一個「model view update」架構](https://github.com/evancz/elm-architecture-tutorial/)，而它的 update 有以下的 signature：`(state, action) => state`。技術上來說，Elm 的「updaters」等同於 Redux 中的 reducers。

跟 Redux 不一樣，Elm 是一個 language，所以它能夠從許多東西獲得好處，像是：強制的 purity、靜態型別、內建 immutability、和模式匹配 (使用 `case` 表達式)。即使你沒有計劃使用 Elm，你也應該閱讀一下 Elm 的架構，並玩玩看它。有一個有趣的 [JavaScript library playground 實做了類似的想法](https://github.com/paldepind/noname-functional-frontend-framework)。我們應該在那裡為 Redux 尋找靈感！一個我們能更接近 Elm 的靜態型別的方式是藉由[使用一個漸進式的型別方案像是 Flow](https://github.com/rackt/redux/issues/290)。

### Immutable

[Immutable](https://facebook.github.io/immutable-js) 是一個實作了 persistent data structures 的 JavaScript library。它是高效能的而且有一個慣用的 JavaScript API。

Immutable 和大部份類似的 library 跟 Redux 是互補的。請自由的結合他們一起使用！

**Redux 不在意你*如何*儲存 state—它可以是一個一般物件、一個 Immutable 物件、或任何其他東西。**你可能需要一個 (de)serialization 機制以撰寫 universal 應用程式並從伺服器 hydrating 它們的 state，但除此之外，你可以使用任何*可以支援 immutability* 的資料儲存 library。例如，使用 Backbone 來作為 Redux state 不合常理，因為 Backbone 的 models 是可以變動的。

請記住，即使你的 immutable library 支援 cursors，你不應該在 Redux 應用程式中使用它們。整個 state tree 應該被想成是唯讀的，而你應該使用 Redux 來更新 state，並訂閱它的更新。因此，藉由 cursor 來寫入在 Redux 是不合理的。**如果你唯一的 cursors 使用案例是讓 state tree 和 UI tree 解開耦合，並逐漸的改善 cursors，你應該試試用 selectors 來取代它。**Selectors 是可以組合的 getter functions。查看 [reselect](http://github.com/faassen/reselect) 來了解一個真的很棒且簡潔的可組合 selectors 實作。

### Baobab

[Baobab](https://github.com/Yomguithereal/baobab) 是另一個實作了 immutable API 用來更新一般的 JavaScript 物件的知名 library。雖然你可以使用它來結合 Redux，不過一起使用它們只有一點點好處。

大多數 Baobab 提供的功能涉及使用 cursors 來更新資料，但是 Redux 強制讓更新資料的唯一方式是 dispatch 一個 action。因此它們用不同的方式解決同一個問題，而不是互補。

跟 Immutable 不一樣，Baobab 還沒有在背後實作任何特別有效率的資料結構，所以你用它跟 Redux 結合並沒有真的贏得什麼。在這個情況下直接使用一般物件會更簡單。

### Rx

[Reactive Extensions](https://github.com/Reactive-Extensions/RxJS) (以及他們正在進行的[現代化重寫](https://github.com/ReactiveX/RxJS)) 是一個極好的方法來管理非同步應用程式的複雜性。實際上，[有一個嘗試是去建立一個把人機互動模型化成相互依賴的 observables 的 library](http://cycle.js.org)。

使用 Redux 結合 Rx 有意義嗎？當然！它們一起可以運作的很好。例如，把 Redux store 暴露成一個 observable 很簡單：

```js
function toObservable(store) {
  return {
    subscribe({ onNext }) {
      let dispose = store.subscribe(() => onNext(store.getState()))
      onNext(store.getState())
      return { dispose }
    }
  }
}
```

同樣的，你可以在把不同的非同步 streams 傳遞給 `store.dispatch()` 之前，組合它們以把它們轉換成 actions。

問題是：如果你已經使用了 Rx 的話，你真的需要 Redux 嗎？可能不需要。要[用 Rx 重新實作 Redux](https://github.com/jas-chen/rx-redux) 並不難。有人說使用 Rx `.scan()` method 只需要兩行。很可能是真的！

如果你有疑問，請查看 Redux 的原始碼 (沒有太多事發生在那裡)，以及它的生態系 (例如：[開發者工具](https://github.com/gaearon/redux-devtools))。如果你不太在意它們而想要一路伴隨著 reactive 資料流，你可能會想要探索一些像是 [Cycle](http://cycle.js.org) 的東西來取代，或甚至把它跟 Redux 結合。請讓我們知道後來是如何發展的！
