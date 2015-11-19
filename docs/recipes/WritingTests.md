# 撰寫測試

大部份你撰寫的 Redux 程式碼都是 functions，而且它們之中有很多是 pure 的，它們不需要 mocking 就能簡單測試。

### 設置

我們推薦用 [Mocha](http://mochajs.org/) 作為測試引擎。
注意，因為它運行在 Node 環境中，所以你不會存取到 DOM。

```
npm install --save-dev mocha
```

要結合 [Babel](http://babeljs.io) 一起使用的話，必須把這個加到你的 `package.json` 的 `scripts`：

```js
{
  ...
  "scripts": {
    ...
    "test": "mocha --compilers js:babel-core/register --recursive",
    "test:watch": "npm test -- --watch",
  },
  ...
}
```

然後執行 `npm test` 來跑一次測試，或是 `npm run test:watch` 來在每一次檔案變更時測試。

### Action Creators

在 Redux 中，action creators 是回傳一般物件的 functions。在測試 action creators 的時候，我們想要測試是否呼叫了正確的 action creator，還有是否回傳了正確的 action。

#### 範例

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}
```
可以像這樣測試：

```js
import expect from 'expect'
import * as actions from '../../actions/TodoActions'
import * as types from '../../constants/ActionTypes'

describe('actions', () => {
  it('should create an action to add a todo', () => {
    const text = 'Finish docs'
    const expectedAction = {
      type: types.ADD_TODO,
      text
    }
    expect(actions.addTodo(text)).toEqual(expectedAction)
  })
})
```

### Async Action Creators

針對使用 [Redux Thunk](https://github.com/gaearon/redux-thunk) 或其他的 middleware 的 async action creators，為了測試，完全的 mock Redux store 是最好的。你仍然可以如下面所示使用 [`applyMiddleware()`](../api/applyMiddleware.md) 以及一個 mock store (你可以在 [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) 找到下面的程式碼)。你也可以使用 [nock](https://github.com/pgte/nock) 來 mock HTTP 請求。

#### 範例

```js
function fetchTodosRequest() {
  return {
    type: FETCH_TODOS_REQUEST
  }
}

function fetchTodosSuccess(body) {
  return {
    type: FETCH_TODOS_SUCCESS,
    body
  }
}

function fetchTodosFailure(ex) {
  return {
    type: FETCH_TODOS_FAILURE,
    ex
  }
}

export function fetchTodos() {
  return dispatch => {
    dispatch(fetchTodosRequest())
    return fetch('http://example.com/todos')
      .then(res => res.json())
      .then(json => dispatch(fetchTodosSuccess(json.body)))
      .catch(ex => dispatch(fetchTodosFailure(ex)))
  }
}
```

可以像這樣測試：

```js
import expect from 'expect'
import { applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as actions from '../../actions/counter'
import * as types from '../../constants/ActionTypes'
import nock from 'nock'

const middlewares = [ thunk ]

/**
 * 用 middleware 建立一個 Redux store 的 mock。
 */
function mockStore(getState, expectedActions, done) {
  if (!Array.isArray(expectedActions)) {
    throw new Error('expectedActions should be an array of expected actions.')
  }
  if (typeof done !== 'undefined' && typeof done !== 'function') {
    throw new Error('done should either be undefined or function.')
  }

  function mockStoreWithoutMiddleware() {
    return {
      getState() {
        return typeof getState === 'function' ?
          getState() :
          getState
      },

      dispatch(action) {
        const expectedAction = expectedActions.shift()

        try {
          expect(action).toEqual(expectedAction)
          if (done && !expectedActions.length) {
            done()
          }
          return action
        } catch (e) {
          done(e)
        }
      }
    }
  }

  const mockStoreWithMiddleware = applyMiddleware(
    ...middlewares
  )(mockStoreWithoutMiddleware)

  return mockStoreWithMiddleware()
}

describe('async actions', () => {
  afterEach(() => {
    nock.cleanAll()
  })

  it('creates FETCH_TODOS_SUCCESS when fetching todos has been done', (done) => {
    nock('http://example.com/')
      .get('/todos')
      .reply(200, { todos: ['do something'] })

    const expectedActions = [
      { type: types.FETCH_TODOS_REQUEST },
      { type: types.FETCH_TODOS_SUCCESS, body: { todos: ['do something']  } }
    ]
    const store = mockStore({ todos: [] }, expectedActions, done)
    store.dispatch(actions.fetchTodos())
  })
})
```

### Reducers

reducer 應該把 action 應用到先前的 state，然後回傳新的 state，而這就是下面所測試的行為。

#### 範例

```js
import { ADD_TODO } from '../constants/ActionTypes'

const initialState = [
  {
    text: 'Use Redux',
    completed: false,
    id: 0
  }
]

export default function todos(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        {
          id: state.reduce((maxId, todo) => Math.max(todo.id, maxId), -1) + 1,
          completed: false,
          text: action.text
        },
        ...state
      ]

    default:
      return state
  }
}
```
可以像這樣測試：

```js
import expect from 'expect'
import reducer from '../../reducers/todos'
import * as types from '../../constants/ActionTypes'

describe('todos reducer', () => {
  it('should return the initial state', () => {
    expect(
      reducer(undefined, {})
    ).toEqual([
      {
        text: 'Use Redux',
        completed: false,
        id: 0
      }
    ])
  })

  it('should handle ADD_TODO', () => {
    expect(
      reducer([], {
        type: types.ADD_TODO,
        text: 'Run the tests'
      })
    ).toEqual(
      [
        {
          text: 'Run the tests',
          completed: false,
          id: 0
        }
      ]
    )

    expect(
      reducer(
        [
          {
            text: 'Use Redux',
            completed: false,
            id: 0
          }
        ],
        {
          type: types.ADD_TODO,
          text: 'Run the tests'
        }
      )
    ).toEqual(
      [
        {
          text: 'Run the tests',
          completed: false,
          id: 1
        },
        {
          text: 'Use Redux',
          completed: false,
          id: 0
        }
      ]
    )
  })
})
```

### Components

React components 的其中一個優點是它們通常都很小，而且只依賴它們的 props。這使它們容易測試。

首先，我們會先安裝 [React Test Utilities](https://facebook.github.io/react/docs/test-utils.html)：

```
npm install --save-dev react-addons-test-utils
```

為了測試 components，我們寫了一個 `setup()` helper，它會傳遞 stubbed callbacks 作為 props 並使用 [React shallow renderer](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering) 來 renders component。這讓獨立的測試在預期 callbacks 會被呼叫時，可以 assert 是否 callbacks 有被呼叫。

#### 範例

```js
import React, { PropTypes, Component } from 'react'
import TodoTextInput from './TodoTextInput'

class Header extends Component {
  handleSave(text) {
    if (text.length !== 0) {
      this.props.addTodo(text)
    }
  }

  render() {
    return (
      <header className='header'>
          <h1>todos</h1>
          <TodoTextInput newTodo={true}
                         onSave={this.handleSave.bind(this)}
                         placeholder='What needs to be done?' />
      </header>
    )
  }
}

Header.propTypes = {
  addTodo: PropTypes.func.isRequired
}

export default Header
```

可以像這樣測試：

```js
import expect from 'expect'
import React from 'react'
import TestUtils from 'react-addons-test-utils'
import Header from '../../components/Header'
import TodoTextInput from '../../components/TodoTextInput'

function setup() {
  let props = {
    addTodo: expect.createSpy()
  }

  let renderer = TestUtils.createRenderer()
  renderer.render(<Header {...props} />)
  let output = renderer.getRenderOutput()

  return {
    props,
    output,
    renderer
  }
}

describe('components', () => {
  describe('Header', () => {
    it('should render correctly', () => {
      const { output } = setup()

      expect(output.type).toBe('header')
      expect(output.props.className).toBe('header')

      let [ h1, input ] = output.props.children

      expect(h1.type).toBe('h1')
      expect(h1.props.children).toBe('todos')

      expect(input.type).toBe(TodoTextInput)
      expect(input.props.newTodo).toBe(true)
      expect(input.props.placeholder).toBe('What needs to be done?')
    })

    it('should call addTodo if length of text is greater than 0', () => {
      const { output, props } = setup()
      let input = output.props.children[1]
      input.props.onSave('')
      expect(props.addTodo.calls.length).toBe(0)
      input.props.onSave('Use Redux')
      expect(props.addTodo.calls.length).toBe(1)
    })
  })
})
```

#### 修復壞掉的 `setState()`

Shallow rendering 現在[如果呼叫 `setState` 會拋出一個錯誤](https://github.com/facebook/react/issues/4019)。React 貌似預期你有用 `setState` 時，DOM 是可以使用的。為了避開這個問題，我們使用 jsdom 讓 React 在 DOM 不能使用時也不會拋出 exception。以下是[設置它](https://github.com/facebook/react/issues/5046#issuecomment-146222515)的方法：

```
npm install --save-dev jsdom
```

接著在你的測試目錄中建立一個 `setup.js` 檔案：

```js
import { jsdom } from 'jsdom'

global.document = jsdom('<!doctype html><html><body></body></html>')
global.window = document.defaultView
global.navigator = global.window.navigator
```

讓這段程式碼在 React 被 import *之前*執行非常重要。為了確保這件事，調整你的 `mocha` 指令在 `package.json` 的選項中加入 `--require ./test/setup.js`：

```js
{
  ...
  "scripts": {
    ...
    "test": "mocha --compilers js:babel/register --recursive --require ./test/setup.js",
  },
  ...
}
```

### 已連結的 Components

如果你使用一個類似 [React Redux](https://github.com/rackt/react-redux) 的 library，你可能正在使用像是 [`connect()`](https://github.com/rackt/react-redux#connectmapstatetoprops-mapdispatchtoprops-mergeprops) 之類的 [higher-order components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)。它讓你把 Redux state 注入一個正規的 React component 裡。

試想下面的 `App` component：

```js
import { connect } from 'react-redux'

class App extends Component { /* ... */ }

export default connect(mapStateToProps)(App)
```

在單元測試中，你通常會像這樣 import `App` component：

```js
import App from './App'
```

但是當你 import 它時，你實際上拿到的是 `connect()` 回傳的包裝過後的 component，而不是 `App` component 本身。如果你想要測試它與 Redux 的互動，這是個好消息：你可以把它跟特別為這個單元測試建立的 store 包在一個 [`<Provider>`](https://github.com/rackt/react-redux#provider-store) 中。但是有時你只是想要測試 component 的 rendering，而不想測試 Redux store。

為了能夠不處理 decorator 即可測試 App component 本身，我們也建議你 export 沒有被 decorated 的 component：

```js
import { connect } from 'react-redux'

// 使用 named export 處理未連結的 component (測試用)
export class App extends Component { /* ... */ }

// 使用 default export 來處理已連結的 component (應用程式用)
export default connect(mapDispatchToProps)(App)
```

因為 default export 仍然是個 decorated component，上面出現的 import 語句會像之前一樣運作，所以你不需要變動應用程式中的程式碼。不過，你現在可以在你的測試檔像這樣 import 沒有被 decorate 的 `App` components：

```js
// 注意大括號：抓取 named export 而不是 default export
import { App } from './App'
```

而如果你兩個都需要：

```js
import ConnectedApp, { App } from './App'
```

在應用程式裡面，你仍然可以像一般一樣 import 它：

```js
import App from './App'
```

你應該只會在測試使用 named export。

>##### 關於混合使用 ES6 模組和 CommonJS 的附註

>如果你有在應用程式的原始碼裡面使用 ES6，不過是用 ES5 撰寫你的測試，你必須知道 Babel 處理了 ES6 `import` 和 CommonJS `require` 之間的互相使用，透過它的 [interop](http://babeljs.io/docs/usage/modules/#interop) 功能就可以讓兩種模組格式一起運作，但行為[有一點點不一樣](https://github.com/babel/babel/issues/2047)。如果你在 default export 旁邊添加了第二個 export，。你就不再能直接使用 `require('./App')` 來 import default。而你必須使用 `require('./App').default`。

### Middleware

Middleware functions 包裝了 Redux 中 `dispatch` 呼叫的行為，所以要測試這個修改後的行為我們必須 mock `dispatch` 呼叫的行為。

#### 範例

```js
import expect from 'expect'
import * as types from '../../constants/ActionTypes'
import singleDispatch from '../../middleware/singleDispatch'

const createFakeStore = fakeData => ({
  getState() {
    return fakeData
  }
})

const dispatchWithStoreOf = (storeData, action) => {
  let dispatched = null
  const dispatch = singleDispatch(createFakeStore(storeData))(actionAttempt => dispatched = actionAttempt)
  dispatch(action)
  return dispatched
};

describe('middleware', () => {
  it('should dispatch if store is empty', () => {
    const action = {
      type: types.ADD_TODO
    }

    expect(
      dispatchWithStoreOf({}, action)
    ).toEqual(action)
  })

  it('should not dispatch if store already has type', () => {
    const action = {
      type: types.ADD_TODO
    }

    expect(
      dispatchWithStoreOf({
        [types.ADD_TODO]: 'dispatched'
      }, action)
    ).toNotExist()
  })
})
```

### 術語表

- [React Test Utils](http://facebook.github.io/react/docs/test-utils.html)：React 的測試 Utilities。

- [jsdom](https://github.com/tmpvar/jsdom)：一個 DOM API 的純 JavaScript 實作。jsdom 讓我們可以不使用瀏覽器就能執行測試。

- [Shallow rendering](http://facebook.github.io/react/docs/test-utils.html#shallow-rendering)：Shallow rendering 讓你可以實體化一個 component 並取得它的 `render` method 的回傳結果，它只會 render 一層的深度而不會遞迴地把 components render 成 DOM。shallow rendering 的結果是一個 [ReactElement](https://facebook.github.io/react/docs/glossary.html#react-elements)。這表示可以存取它的 children、props 並測試它是否如預期運作。這也意味你改變 child component 不會影響到 parent component 的測試。
