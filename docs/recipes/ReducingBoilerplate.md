# 減少 Boilerplate

Redux 的一部分是[受到 Flux 啟發](../introduction/PriorArt.md)，而 Flux 最常見的抱怨是它使你必須寫一大堆的 boilerplate。在這份 recipe 中，我們將會思考 Redux 如何讓我們依照個人風格、團隊喜好、長期可維護性，等等，來選擇我們希望程式碼要多冗長。

## Actions

Actions 是描述應用程式中生了什麼事的一般物件，並且作為描述改變資料意圖的唯一方式。重要的是，**你必須 dispatch 的這些 actions 物件並不是 boilerplate，而是 Redux 的[基本設計決策](../introduction/ThreePrinciples.md)其中之一**。

有一些框架宣稱與 Flux 類似，但沒有 action 物件的觀念。在可預測性方面，這是從 Flux 或是 Redux 的一種退步。如果沒有可以 serialize 的一般物件 actions，就不可能紀錄並重播使用者的操作狀態，或是實現 [hot reloading 與時間旅行](https://www.youtube.com/watch?v=xsSnOQynTHs)。如果你仍然希望直接改變資料，那你不需要使用 Redux。

Actions 看起來像這樣：

```js
{ type: 'ADD_TODO', text: 'Use Redux' }
{ type: 'REMOVE_TODO', id: 42 }
{ type: 'LOAD_ARTICLE', response: { ... } }
```

actions 通常會有一個常數的 type 屬性來幫助 reducers (或是 Flux 裡面的 Stores) 來辨識它們，這是一個常見的慣例。我們建議你使用字串而不要使用 [Symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 來當作 action types，因為字串可以 serialize，而使用 Symbols 的話，你會讓記錄和重播比想像中更困難。

在傳統 Flux 中，你會把每個 action type 定義為一個字串常數：


```js
const ADD_TODO = 'ADD_TODO'
const REMOVE_TODO = 'REMOVE_TODO'
const LOAD_ARTICLE = 'LOAD_ARTICLE'
```

為什麼這個是有好處的呢？**常數常常被認為是不必要的，而對小專案來說，這或許是對的。**但對大一點的專案來說，把 action types 定義成常數有一些好處：

* 它有助於保持命名的一致性，因為所有的 action types 都被聚集在同一個地方。
* 有時候你會想要在你開始做新功能之前，先看到所有存在的 actions。有可能你需要的 action 已經被團隊其他人加進來了，但你並不知道。
* 在 Pull Request 裡面被添加、移除、變更的 action types 清單，有助於團隊裡的每一個人跟上現在的進度並實作新功能。
* 如果你在 import 一個 action 常數時打錯字，你會得到 `undefined`。Redux 會在 dispatch 這樣的 action 時立刻拋出錯誤，你將會很快找到錯誤。

要在你的專案選擇怎樣的慣例完全取決於你。你可以從使用行內字串開始，在之後轉換成使用常數，在更後面可以把他們分進一個單獨的檔案。Redux 在這裡沒有任何意見，所以請使用你的最佳判斷。

## Action Creators

另一種常見的慣例是，不要在你 dispatch actions 的地方建立行內 action 物件，而是建立 functions 產生它們。

例如，取代這個使用字面物件的 `dispatch` 呼叫：

```js
// 在某處的 event handler 中
dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
```

你可以在一個單獨的檔案中撰寫一個 action creator，並從你的 component import 它：

#### `actionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}
```

#### `AddTodo.js`

```js
import { addTodo } from './actionCreators'

// 在某處的 event handler 中
dispatch(addTodo('Use Redux'))
```

Action creators 常常被批評為 boilerplate。沒關係，你並不需要撰寫他們！**如果你覺得字面物件比較適合你的專案，你也可以使用它。**然而，撰寫 action creators 有一些好處你應該要知道。

比如說，設計師在檢視了我們的原型之後回來找我們，並告訴我們需要限制三個的 todos 上限。我們可以藉由把我們的 action creator 用 [redux-thunk](https://github.com/gaearon/redux-thunk) middleware 重寫成 callback 形式，並添加一個提早中斷來達成這個：

```js
function addTodoWithoutCheck(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function addTodo(text) {
  // 這種形式是被 Redux Thunk middleware 所允許，
  // 描述在下面的「非同步的 Action Creators」章節。
  return function (dispatch, getState) {
    if (getState().todos.length === 3) {
      // 提早中斷
      return
    }

    dispatch(addTodoWithoutCheck(text))
  }
}
```
我們只是修改了 `addTodo` action creator 如何產生行為，呼叫的程式碼完全看不到差別。**我們不需要擔心需要查看每個添加 todos 的地方，來確保它們有做這個檢查。**Action creators 讓你解開了 dispatch action 的額外邏輯與 components 實際發送這些 actions 之間的耦合。在應用程式處於很積極開發且需求常常改變的狀態下這非常方便。

### 產生 Action Creators

一些像是 [Flummox](https://github.com/acdlite/flummox) 的框架會自動從 action creator function 的定義產生 action type 常數。它的想法是你不需要同時定義 `ADD_TODO` 常數和 `addTodo()` action creator。在這背後，這樣的方法還是會產生 action type 常數，不過它們是暗地中被建立，所以這是一個間接層並可能導致困惑。我們建議明確地建立你的 action type 常數。

撰寫簡單的 action creators 可能很煩人並常常最後產生多餘的 boilerplate 程式碼：

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function editTodo(id, text) {
  return {
    type: 'EDIT_TODO',
    id,
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

你還是可以撰寫一個產生 action creator 的 function：

```js
function makeActionCreator(type, ...argNames) {
  return function(...args) {
    let action = { type }
    argNames.forEach((arg, index) => {
      action[argNames[index]] = args[index]
    })
    return action
  }
}

const ADD_TODO = 'ADD_TODO'
const EDIT_TODO = 'EDIT_TODO'
const REMOVE_TODO = 'REMOVE_TODO'

export const addTodo = makeActionCreator(ADD_TODO, 'todo')
export const editTodo = makeActionCreator(EDIT_TODO, 'id', 'todo')
export const removeTodo = makeActionCreator(REMOVE_TODO, 'id')
```
也有一些 utility libraries 可以幫助產生 action creators，像是 [redux-action-utils](https://github.com/insin/redux-action-utils) 和 [redux-actions](https://github.com/acdlite/redux-actions)。這些可以幫助減少你的 boilerplate 程式碼並遵守像是 [Flux Standard Action (FSA)](https://github.com/acdlite/flux-standard-action) 的標準。

## 非同步的 Action Creators

[Middleware](../Glossary.html#middleware) 讓你注入自訂的邏輯，在每一個 action 物件被 dispatch 之前轉譯它。Async actions 是 middleware 最常見的使用案例。

沒有任何 middleware 的話，[`dispatch`](../api/Store.md#dispatch) 只接受一個一般的物件，所以我們必須在 components 裡面執行 AJAX 呼叫：

#### `actionCreators.js`

```js
export function loadPostsSuccess(userId, response) {
  return {
    type: 'LOAD_POSTS_SUCCESS',
    userId,
    response
  }
}

export function loadPostsFailure(userId, error) {
  return {
    type: 'LOAD_POSTS_FAILURE',
    userId,
    error
  }
}

export function loadPostsRequest(userId) {
  return {
    type: 'LOAD_POSTS_REQUEST',
    userId
  }
}
```

#### `UserInfo.js`

```js
import { Component } from 'react'
import { connect } from 'react-redux'
import { loadPostsRequest, loadPostsSuccess, loadPostsFailure } from './actionCreators'

class Posts extends Component {
  loadData(userId) {
    // 藉由 React Redux `connect()` 呼叫注入進去 props：
    let { dispatch, posts } = this.props

    if (posts[userId]) {
      // 這是已經被快取的資料！不要做任何事情。
      return
    }

    // Reducer 可以藉由設定 `isFetching` 來應對這個 action，
    // 並因此讓我們可以顯示一個 spinner。
    dispatch(loadPostsRequest(userId))

    // Reducer 可以藉由填入`users` 來應對這些 actions。
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch(loadPostsSuccess(userId, response)),
      error => dispatch(loadPostsFailure(userId, error))
    )
  }

  componentDidMount() {
    this.loadData(this.props.userId)
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.loadData(nextProps.userId)
    }
  }

  render() {
    if (this.props.isFetching) {
      return <p>Loading...</p>
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    )

    return <div>{posts}</div>
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts)
```

不過，這很快就會重複，因為不同的 components 會需要從一樣的 API 端點請求資料。除此之外，我們想要從許多的 components 來重用一部分的邏輯 (例如，當有被快取的資料可以使用時提早中斷)。

**Middleware 讓我們撰寫更有表達性、有能力非同步的 action creators。**它讓我們能 dispatch 一些不是一般物件的東西，並轉譯它的值。例如，middleware 可以「catch」被 dispatch 的 Promises，並把他們轉換成成對的請求和成功/失敗的 actions。

最簡單的 middleware 例子就是 [redux-thunk](https://github.com/gaearon/redux-thunk)。**「Thunk」 middleware 讓你把 action creators 寫成「thunks」，它就是一個回傳 functions 的 functions。**這反轉了控制：你會拿到 `dispatch` 作為一個參數，所以你可以寫一個 dispatch 很多次的 action creator。

>##### 附註

>Thunk middleware 只是一個 middleware 的例子。Middleware 不是「讓你 dispatch functions」：它讓你 dispatch 任何你使用的特定 middleware 知道要如何處理的東西。Thunk middleware 在你 dispatch functions 的時候添加了一個特定的行為，不過實際上這取決於你使用的 middleware。

試想把上面的程式碼用 [redux-thunk](https://github.com/gaearon/redux-thunk) 來重寫：

#### `actionCreators.js`

```js
export function loadPosts(userId) {
  // 被 thunk middleware 所轉譯：
  return function (dispatch, getState) {
    let { posts } = getState()
    if (posts[userId]) {
      // 這是已經被快取的資料！不要做任何事情。
      return
    }

    dispatch({
      type: 'LOAD_POSTS_REQUEST',
      userId
    })

    // 非同步的 Dispatch 原生的 actions
    fetch(`http://myapi.com/users/${userId}/posts`).then(
      response => dispatch({
        type: 'LOAD_POSTS_SUCCESS',
        userId,
        response
      }),
      error => dispatch({
        type: 'LOAD_POSTS_FAILURE',
        userId,
        error
      })
    )
  }
}
```

#### `UserInfo.js`

```js
import { Component } from 'react'
import { connect } from 'react-redux'
import { loadPosts } from './actionCreators'

class Posts extends Component {
  componentDidMount() {
    this.props.dispatch(loadPosts(this.props.userId))
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.userId !== this.props.userId) {
      this.props.dispatch(loadPosts(nextProps.userId))
    }
  }

  render() {
    if (this.props.isFetching) {
      return <p>Loading...</p>
    }

    let posts = this.props.posts.map(post =>
      <Post post={post} key={post.id} />
    )

    return <div>{posts}</div>
  }
}

export default connect(state => ({
  posts: state.posts
}))(Posts)
```

這樣打的字更少了！如果你想要，你還是可以使用「原生的」action creators 像是 `loadPostsSuccess`，你會從 container 的 `loadPosts` action creator 中來使用它。

**最後，你可以寫自己的 middleware。**比如说，你想要歸納前面的模式並用像這樣的方式來描述非同步的 action creators：

```js
export function loadPosts(userId) {
  return {
    // 要在之前和之後發送的 actions types
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    // 檢查快取 (可選擇的):
    shouldCallAPI: (state) => !state.users[userId],
    // 執行抓取資料：
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    // 要在開始/結束 actions 注入的參數
    payload: { userId }
  }
}
```

可以轉譯這些 actions 的 middleware 看起來像這樣：

```js
function callAPIMiddleware({ dispatch, getState }) {
  return function (next) {
    return function (action) {
      const {
        types,
        callAPI,
        shouldCallAPI = () => true,
        payload = {}
      } = action

      if (!types) {
        // 普通的 action：把它傳遞下去
        return next(action)
      }

      if (
        !Array.isArray(types) ||
        types.length !== 3 ||
        !types.every(type => typeof type === 'string')
      ) {
        throw new Error('Expected an array of three string types.')
      }

      if (typeof callAPI !== 'function') {
        throw new Error('Expected fetch to be a function.')
      }

      if (!shouldCallAPI(getState())) {
        return
      }

      const [ requestType, successType, failureType ] = types

      dispatch(Object.assign({}, payload, {
        type: requestType
      }))

      return callAPI().then(
        response => dispatch(Object.assign({}, payload, {
          response: response,
          type: successType
        })),
        error => dispatch(Object.assign({}, payload, {
          error: error,
          type: failureType
        }))
      )
    }
  }
}
```

一旦把它傳遞到 [`applyMiddleware(...middlewares)`](../api/applyMiddleware.md) 之後，你就可以用同樣的方式撰寫全部的 API 呼叫 action creators：

```js
export function loadPosts(userId) {
  return {
    types: ['LOAD_POSTS_REQUEST', 'LOAD_POSTS_SUCCESS', 'LOAD_POSTS_FAILURE'],
    shouldCallAPI: (state) => !state.users[userId],
    callAPI: () => fetch(`http://myapi.com/users/${userId}/posts`),
    payload: { userId }
  }
}

export function loadComments(postId) {
  return {
    types: ['LOAD_COMMENTS_REQUEST', 'LOAD_COMMENTS_SUCCESS', 'LOAD_COMMENTS_FAILURE'],
    shouldCallAPI: (state) => !state.posts[postId],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`),
    payload: { postId }
  }
}

export function addComment(postId, message) {
  return {
    types: ['ADD_COMMENT_REQUEST', 'ADD_COMMENT_SUCCESS', 'ADD_COMMENT_FAILURE'],
    callAPI: () => fetch(`http://myapi.com/posts/${postId}/comments`, {
      method: 'post',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ message })
    }),
    payload: { postId, message }
  }
}
```

## Reducers

Redux 透過把更新邏輯描述成 function 來減少許多 Flux stores 的 boilerplate。function 比物件更簡單 ，也比 class 更簡單。

試想這個 Flux store：

```js
let _todos = []

const TodoStore = Object.assign({}, EventEmitter.prototype, {
  getAll() {
    return _todos
  }
})

AppDispatcher.register(function (action) {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      let text = action.text.trim()
      _todos.push(text)
      TodoStore.emitChange()
  }
})

export default TodoStore
```

藉由 Redux，一樣的更新邏輯可以被描述成一個 reducing function：

```js
export function todos(state = [], action) {
  switch (action.type) {
  case ActionTypes.ADD_TODO:
    let text = action.text.trim()
    return [ ...state, text ]
  default:
    return state
  }
}
```

`switch` 語句*不*算真實的 boilerplate。真實的 Flux boilerplate 是概念性的：需要發送更新、需要註冊 Store 到 Dispatcher、Store 需要是一個物件 (並在你想要一個 universal 應用程式的時候出現併發症)。

不幸的事，許多人仍然依照它是不是在文件中使用 `switch` 語句來選擇框架。如果你不喜歡 `switch`，你可以用一個單一的 function 來解決這個問題，就像我們下面所展示的。

### 產生 Reducers

讓我們來寫一個 function，它可以讓我們把 reducers 表達成一個從 action types 到 handler 的物件映射。例如，如果我們想要像這樣定義 `todos` reducers：

```js
export const todos = createReducer([], {
  [ActionTypes.ADD_TODO](state, action) {
    let text = action.text.trim()
    return [ ...state, text ]
  }
})
```

我們可以撰寫下面的 helper 來完成這個：

```js
function createReducer(initialState, handlers) {
  return function reducer(state = initialState, action) {
    if (handlers.hasOwnProperty(action.type)) {
      return handlers[action.type](state, action)
    } else {
      return state
    }
  }
}
```

這並不難，對吧？Redux 不預設提供這個 helper function，因為有太多方式可以實作它了。你可能會希望它能自動的把一般 JS 物件轉換成 Immutable 物件來 hydrate 來自伺服器的 state。你可能會希望把被回傳的 state 和當下的 state 合併。可能有許多不同的方法可以實作一個「catch all」handler。這一切都取決於你為團隊在特定專案所選擇的慣例。

Redux 的 reducer API 是 `(state, action) => state`，不過你要如何建立這些 reducers 完全取決於你。
