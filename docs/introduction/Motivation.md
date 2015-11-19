# 動機

隨著對 JavaScript single-page 應用程式的要求變得越來愈複雜，**我們的程式碼必須管理高於以往的 state**。這些 state 可以包括伺服器回應和快取的資料，以及本地端建立而尚未保存到伺服器的資料。UI state 也越來愈複雜，因為我們需要管理 active route、被選擇的 tab、是否要顯示 spinner、 pagination 控制應不應該被顯示，等等。

管理這個不斷變化的 state 是很困難的。如果一個 model 可以更新其他的 model，接著一個 view 可以更新一個 model，而它更新了另一個 model，而這個順帶的，可能造成另一個 view 被更新。到了某個時間點，你不再了解你的應用程式中發生了些什麼，因為你已經**失去對 state 何時、為什麼、如何運作的控制權。**當系統不透明且充滿不確定性，就很難去重現 bugs 或添加新的功能。

彷彿這樣還不夠糟糕，試想一些**在前端產品開發越來越普遍的新需求**。作為開發者，我們被期望要去處理 optimistic updates、伺服器端 render、在轉換 route 前抓取資料，等等。我們發現自己嘗試去管理一個從來沒有處理過的複雜度，我們不免會有這樣的疑問：[是時候該放棄了嗎？](http://www.quirksmode.org/blog/archives/2015/07/stop_pushing_th.html) 答案是_不_。

這樣的複雜度很難去處理，因為 **我們混合了兩個概念**，它們都是人類的頭腦非常難去思考的概念：**變更和非同步。**我稱它們為 [Mentos and Coke](https://en.wikipedia.org/wiki/Diet_Coke_and_Mentos_eruption)。分開都好好的，但混在一起就變成一團糟。[React](http://facebook.github.io/react) 之類的 Libraries 試圖藉由移除非同步和直接的 DOM 操作，來在 view layer 解決這個問題。但是，管理資料 state 的部分留下來讓你自己決定。這就是 Redux 的切入點。

跟隨著 [Flux](http://facebook.github.io/flux)、[CQRS](http://martinfowler.com/bliki/CQRS.html) 和 [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) 的腳步，Redux 藉由強加某些限制在更新發生的方式和時機上，**試圖讓 state 的變化更有可預測性**。這些限制都反應在 Redux 的[三大原則](ThreePrinciples.md)中。
