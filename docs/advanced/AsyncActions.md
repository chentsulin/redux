# Async Action

在[基礎教學](../basics/README.md)中，我們建立了一個簡單的 todo 應用程式。它完全是同步的。每次 action 被 dispatch，state 都會立刻被更新。

在這份教學中，我們將會建立一個不同而且非同步的應用程式。它將會使用 Reddit API 針對選擇的 subreddit 來顯示現在的頭條新聞。如何讓非同步與 Redux 資料流結合呢？

## Action

當你呼叫一個非同步 API，有兩個關鍵的時刻：你開始呼叫的的時候，以及當你收到回應 (或是逾時) 的時候。

這兩個時刻通常都可以要求改變應用程式的 state；要做到這一點，你需要 dispatch 會被 reducer 同步處理的一般 action。通常，針對任何一個 API 請求你會需要 dispatch 至少三個不同種類的 action：

* **一個告知 reducer 請求開始的 action。**

  reducer 可以藉由打開 state 裡的 `isFetching` flag 來處理這個 action。這樣 UI 就知道是時候顯示一個 spinner 了。

* **一個告知 reducer 請求成功完成的 action。**

  reducer 可以藉由把新的資料合併到它們管理的 state 裡並重置 `isFetching` 屬性來處理這個 action。UI 將會把 spinner 隱藏，並顯示抓回來的資料。

* **一個告知 reducer 請求失敗的 action。**

  Reducers 可以藉由重置 `isFetching` 屬性來處理這個 action。此外，有些 reducer 也會想要儲存錯誤訊息，這樣 UI 就可以顯示它。

你可以在 actions 裡使用一個專用的 `status` 屬性：

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

或者你也可以為它們定義不同的 types：

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

無論要選擇使用單一一個 action type 與 flag，或是選擇多種 action type，這都取決於你。這是一個你需要跟你的團隊一起決定的慣例。多種的 type 出現錯誤的空間更小，不過如果你使用像是 [redux-actions](https://github.com/acdlite/redux-actions) 之類的 helper library 來產生 action creator 和 reducer 的話，這不會是個問題。

無論你選擇怎樣的慣例，請在整個應用程式中貫徹下去。
在這份教學中，我們將會使用不同的 type。

## 同步的 Action Creator

讓我們開始定義幾個在範例應用程式中需要的同步 action type 和 action creator。在這裡，使用者可以選擇顯示一個 subreddit：

#### `actions.js`

```js
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```

他們也可以按下「刷新」按鈕來更新它：

```js
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```

有一些 action 是藉由使用者互動來控制。我們也會有其他種類藉由網路請求控制的 action。我們之後將會看到要如何 dispatch 它們，但現在，我們只想要定義它們。

當是時候針對 subreddit 抓取 posts 時，我們會 dispatch 一個 `REQUEST_POSTS` action：

```js
export const REQUEST_POSTS = 'REQUEST_POSTS'

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```

把 `SELECT_SUBREDDIT` 和 `INVALIDATE_SUBREDDIT` 分開對它來說是非常重要的。當它們可能一個發生在另一個之後，而隨著應用程式變得更複雜，你可能會想要針對使用者的動作獨立的抓取一些資料 (舉例來說，預先抓取最有人氣的 subreddits，或是在一段時間之後刷新舊的資料)。你可能還需要對應 route 的改變去抓取資料，所以在初期就把抓取資料跟 一些特定的 UI 事件耦合在一起不是很明智。

最後，當網路請求傳回來時，我們會 dispatch `RECEIVE_POSTS`：

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

這些就是我們現在需要知道的。可以隨著網路請求一併 dispatch 這些 action 的特別機制將會在之後討論。

>##### 關於錯誤處理的附註

>在一個真實的應用程式中，你也會想要在請求失敗時 dispatch 一個 action。我們不會在這份教學中實作錯誤處理，不過 [real world example](../introduction/Examples.md#real-world) 展示了其中一個可行的方法。

## 設計 State 的形狀

就像在基礎教學中一樣，你會需要在倉促的開始實作之前，先[設計應用程式 state 的形狀](../basics/Reducers.md#designing-the-state-shape)。有了非同步的程式碼，會有更多 state 要關照，所以我們需要把它完整思考過一遍。

這個部分常常讓初學者困惑，因為要用什麼資訊來描述一個非同步的應用程式的 state，還有要如何在單一一個 tree 上組織它不是一開始就很明顯。

我們會先從最常見的使用案例開始：清單。網頁應用程式時常會顯示一些東西的清單。例如：post 的清單、朋友的清單。 你會需要弄清楚你的應用程式可以顯示什麼類型的清單。你想要把它們分別儲存在 state 裡，因為這樣你可以快取它們而且只在需要時才重新抓取資料。

這就是我們的「Reddit 頭條新聞」應用程式的 state 形狀可能會看起來的樣子：

```js
{
  selectedSubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```

這裡有幾個重要的點：

* 我們分別的儲存每一個 subreddit 的資訊，所以我們可以快取每一個 subreddit。當使用者第二次在它們之間切換，將會即時更新，而且除非我們想要不然我們不需要重新抓取資料。不要擔心這些東西全部都會在記憶體裡：除非你正在處理數以萬計的項目，並且你的使用者不太關閉 tab，不然你完全不需要用任何方式清除他們。

* 針對每一個項目清單，你會想儲存一個 `isFetching` 屬性來顯示 spinner，`didInvalidate` 讓你可以在資料已經過時的時候再去觸發更新它，`lastUpdated` 讓你知道最後一次抓取資料的時間，還有 `items` 它們自己。在一個真實的應用程式中，你也會想要儲存 pagination state 像是 `fetchedPageCount` 和 `nextPageUrl`。

>##### 關於巢狀 Entity 的附註

>在這個範例中，我們把收到的項目跟 pagination 資訊儲存在一起。但是，如果你有巢狀且互相參考的 entity，或是如果你讓使用者可以編輯項目，那這個方法不會運作得很好。試想如果使用者想要去編輯一個抓回來的 post，但是這個 post 被複製到 state tree 中的好幾個地方。實作這個將會非常痛苦。

>如果你有巢狀的 entity，或是如果你讓使用者可以編輯接收到的項目，你應該把它們分別保存在 state 中，就像它是一個資料庫。在 pagination 資訊中，你只會藉由它們的 IDs 來參考它們。這使你能讓它們始終保持更新到最新狀態。[real world example](../introduction/Examples.md#real-world) 展示了這個方法，並使用了 [normalizr](https://github.com/gaearon/normalizr) 來正規化巢狀的 API 回應。用這個方法，你的 state 可能會看起來像這樣：

>```js
> {
>   selectedSubreddit: 'frontend',
>   entities: {
>     users: {
>       2: {
>         id: 2,
>         name: 'Andrew'
>       }
>     },
>     posts: {
>       42: {
>         id: 42,
>         title: 'Confusion about Flux and Relay',
>         author: 2
>       },
>       100: {
>         id: 100,
>         title: 'Creating a Simple Application Using React JS and Flux Architecture',
>         author: 2
>       }
>     }
>   },
>   postsBySubreddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [ 42, 100 ]
>     }
>   }
> }
>```

>在這份教學中，我們不會把 entity 正規化，不過針對一個更動態的應用程式，你應該考慮這樣做。

## 處理 Actions

在走進把 dispatch action 和網路請求結合的細節之前，我們將會撰寫 reducer 給我們上面定義的 action。

>##### 關於 Reducer Composition 的附註

> 這裡，我們假設你已經了解藉由 [`combineReducers()`](../api/combineReducers.md) 來做 reducer composition，它被描述在[基礎教學](../basics/README.md)的[拆分 Reducer](../basics/Reducers.md#splitting-reducers) 章節中。如果你不了解，請[先閱讀它](../basics/Reducers.md#splitting-reducers)。

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedSubreddit(state = 'reactjs', action) {
  switch (action.type) {
    case SELECT_SUBREDDIT:
      return action.subreddit
    default:
      return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = {}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedSubreddit
})

export default rootReducer
```

在這份程式碼中，有兩個有趣的部分：

* 我們使用 ES6 computed property 語法，所以我們可以把 `state[action.subreddit]` 和 `Object.assign()` 更新成一個更簡潔的方式。這樣：

  ```js
  return Object.assign({}, state, {
    [action.subreddit]: posts(state[action.subreddit], action)
  })
  ```
  等同於：

  ```js
  let nextState = {}
  nextState[action.subreddit] = posts(state[action.subreddit], action)
  return Object.assign({}, state, nextState)
  ```
* 我們把 `posts(state, action)` 抽出來管理具體的 post 清單的 state。這只是 [reducer composition](../basics/Reducers.md#splitting-reducers)！這是我們選擇用來把 reducer 拆分成更小的 reducers 的方式，而在這個案例中，我們把在物件中更新項目的工作委派給 `posts` reducer。[real world example](../introduction/Examples.md#real-world) 更進一步，展示了如何建立一個 reducer factory 來參數化 pagination reducer。

請記得 reducer 只是些 function，所以你可以盡你所能舒適的使用 functional composition 和 higher-order function。

## 非同步的 Action Creator

最後，我們要如何一起使用我們[之前定義的](#synchronous-action-creators)同步的 action creator 和網路請求呢？用 Redux 要做到這個的標準方式是使用 [Redux Thunk middleware](https://github.com/gaearon/redux-thunk)。它屬於一個獨立的套件，叫做 `redux-thunk`。我們[晚點](Middleware.md)會解釋 middleware 一般來說是如何運作的；現在，你只有一件重要的事必需知道：藉由使用這個特定的 middleware，action creator 可以回傳一個 function 來取代 action 物件。這樣的話，function creator 就變成一個 [thunk](https://en.wikipedia.org/wiki/Thunk)。

當一個 action creator 回傳一個 function 的時候，這個 function 將會被 Redux Thunk middleware 執行。這個 function 不需要是 pure 的；因此它被允許一些有 side effect 的動作，包括執行非同步的 API 呼叫。這個 function 也可以 dispatch action—像是那些我們之前定義的同步 action。

我們仍然可以把 這些特別的 thunk action creator 定義在我們的 `actions.js` 檔案中：

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

// 迎接我們的第一個 thunk action creator！
// 雖然它裡面不一樣，不過你可以就像其他的 action creator 一般使用它：
// store.dispatch(fetchPosts('reactjs'))

function fetchPosts(subreddit) {

  // Thunk middleware 知道如何去處理 function。
  // 它把 dispatch method 作為參數傳遞給 function，
  // 因此讓它可以自己 dispatch action。

  return function (dispatch) {

    // 第一個 dispatch：更新應用程式 state 以告知
    // API 呼叫開始了。

    dispatch(requestPosts(subreddit))

    // 被 thunk middleware 呼叫的 function 可以回傳一個值，
    // 那會被傳遞作為 dispatch method 的回傳值。

    // 在這個案例中，我們回傳一個 promise 以等待。
    // 這不是 thunk middleware 所必須的，不過這樣對我們來說很方便。

    return fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // 我們可以 dispatch 許多次！
        // 在這裡，我們用 API 呼叫的結果來更新應用程式的 state。

        dispatch(receivePosts(subreddit, json))
      )

      // 在一個真實世界中的應用程式，你也會想要
      // 捕捉任何網路呼叫中的錯誤。
  }
}
```

>##### 關於 `fetch` 的附註

>在範例中，我們使用 [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)。它是一個用來建立網路請求的新 API，取代 `XMLHttpRequest` 最常見的需求。因為大部份的瀏覽器還沒有原生的支援它，我們建議你使用 [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) library：

>```js
// 在每一個你使用 `fetch` 的檔案做這個
>import fetch from 'isomorphic-fetch'
>```

>內部機制中，它在客戶端上會使用 [`whatwg-fetch` polyfill](https://github.com/github/fetch)，而在伺服器上會使用 [`node-fetch`](https://github.com/bitinn/node-fetch)，所以如果你把應用程式改變成 [universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9) 的，不需要改變任何的 API 呼叫。

>要注意，所有的 `fetch` polyfill 都假設已經有一個 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) polyfill。要確保你有一個 Promise polyfill 的最簡單的方式，是在你的進入點任何其他的程式碼執行之前啟用 Babel 的 ES6 polyfill：

>```js
>// 在你的應用程式任何其他的程式碼之前做一次這個
>import 'babel-polyfill'
>```

我們要如何把 Redux Thunk middleware 加進 dispatch 機制裡？我們使用 Redux 裡的 [`applyMiddleware()`](../api/applyMiddleware.md) store enhancer，如下所示：

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // 讓我們來 dispatch() function
    loggerMiddleware // 巧妙的 middleware，用來 log action
  )
)

store.dispatch(selectSubreddit('reactjs'))
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```

有關 thunk 的好處是，它們可以 dispatch 其他 thunk 的結果：

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`https://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // 記住，function 也會收到 getState()，
  // 它讓你選擇下一個要 dispatch 什麼。

  // 如果被快取的值已經是可用的話，
  // 這對於避免網路請求很有用。

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // 從 thunk Dispatch 一個 thunk！
      return dispatch(fetchPosts(subreddit))
    } else {
      // 讓呼叫的程式碼知道沒有東西要等待了。
      return Promise.resolve()
    }
  }
}
```

這讓我們可以漸漸的撰寫更複雜的非同步控制流程，而使用的程式碼卻可以保持幾乎一樣：

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState())
)
```

>##### 關於伺服器 Render 的附註

>Async action creator 對伺服器 render 特別方便。你可以建立一個 store，dispatch 一個單一的 async action creator，它會 dispatch 其他的 async action creator 來為整個應用程式抓取資料，並在 Promise 回傳並完成之後才 render。接著你 render 之前需要的 state 將必須被 hydrate 到你的 store。

[Thunk middleware](https://github.com/gaearon/redux-thunk) 不是在 Redux 中協調非同步 action 的唯一方式：
- 你可以使用 [redux-promise](https://github.com/acdlite/redux-promise) 或是 [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) 來 dispatch Promise 取代 function。
- 你可以使用 [redux-observable](https://github.com/redux-observable/redux-observable) 來 dispatch Observables。
- 你可以使用 [redux-saga](https://github.com/yelouafi/redux-saga/) middleware 來建置更複雜的非同步 action。
- 你甚至可以撰寫一個客製化的 middleware 來描述你的 API 呼叫，像是 [real world example](../introduction/Examples.md#real-world) 做的那樣。

你可以自由地嘗試幾個選項，選擇一個你喜歡的慣例，並遵守它，無論有沒有使用 middleware。

## 連結到 UI

Dispatch async action 跟 dispatch 同步的 action 沒有什麼不同，所以我們不會詳細討論這個。查看[搭配 React 運用](../basics/UsageWithReact.md)了解有關結合 Redux 與 React component 的介紹。查看[範例：Reddit API](ExampleRedditAPI.md)來取得在這個範例中討論的完整原始碼。

## 下一步

查看[非同步資料流](AsyncFlow.md)回顧一下 async action 如何融入 Redux 資料流。
