# 遷移到 Redux

Redux 不是一個整體的框架，而是一系列的介面和[一些可以讓它們一起合作的 functions](../api/README.md)。甚至你大多數的「Redux 程式碼」都不會用到 Redux APIs，因為你大部份的時間都會是在撰寫 functions。

這讓遷入還是遷出 Redux 都很簡單。
我們不想要把你綁在上面！

## 從 Flux

[Reducers](../Glossary.md#reducer) 擷取了 Flux Stores 的「本質」，所以無論你是使用 [Flummox](http://github.com/acdlite/flummox)、[Alt](http://github.com/goatslacker/alt)、[傳統 Flux](https://github.com/facebook/flux)，或任何其他的 Flux library，漸漸的遷移一個已存在的 Flux 專案到 Redux 都是可以的。

同樣的你也可以反向的依照一樣的步驟，來從 Redux 遷移到上述任何的這些 libraries。

你的流程會看起來像這樣：

* 建立一個叫做 `createFluxStore(reducer)` 的 function，它用來從 reducer function 建立一個相容於你現存應用程式的 Flux store。從 Redux 內部來說，它可能看起來很像 [`createStore`](../api/createStore.md) ([原始碼](https://github.com/rackt/redux/blob/master/src/createStore.js)) 的實作。它的 dispatch handler 應該只針對 action 去呼叫 `reducer`、儲存 state 變化、並發送 change 事件。

* 這讓你可以漸漸的把應用程式中的每一個 Flux Store 改寫成 reducer，但仍然 export `createFluxStore(reducer)`，所以應用程式的剩餘部分並不會察覺到有什麼事發生，也只會看到 Flux stores。

* 當你改寫了你的 Stores 後，你將會發現你需要避免某些 Flux 的反模式，例如：在 Store 裡面抓取 API、或在 Stores 裡面觸發 actions。一旦你把它們改寫成基於 reducers，你的 Flux 程式碼會更容易去了解！

* 當你已經把所有的 Flux Stores 都改成基於 reducers 之上去實作，你可以把 Flux library 置換成一個單一的 Redux store，並藉由 [`combineReducers(reducers)`](../api/combineReducers.md) 來把你已經有的這些 reducers 結合成一個。

* 現在只剩下要移植 UI 來[使用 react-redux](../basics/UsageWithReact.md) 或其他類似的東西。

* 最後，你可能會想要開始使用一些 Redux 特有的功能像是 middleware ，來進一步簡化你的非同步程式碼。

## 從 Backbone

Backbone 的 model 層跟 Redux 有很大的差異，所以我們不建議把它們混在一起。如果可能的話，最好從頭開始重寫你的應用程式的 model 層而不是把 Backbone 連接到 Redux。然而，如果重寫是不可行的，那你可以使用 [backbone-redux](https://github.com/redbooth/backbone-redux) 來漸漸的遷移，並維持 Redux store 跟 Backbone models 和 collections 之間的同步。
