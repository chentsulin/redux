# 搭配 React 運用

在一開始，我們必須強調，Redux 跟 React 並沒有關係。你可以用 React、Angular、Ember、jQuery 或甚至原生 JavaScript 來撰寫 Redux 應用程式。

不過，Redux 與像是 [React](http://facebook.github.io/react/) 和 [Deku](https://github.com/dekujs/deku) 之類的框架一起運作的特別好，因為它們讓你把 UI 描述成一個 state 的 function，而 Redux 對應 actions 來發出 state 更新。

我們將會使用 React 來建置我們的簡易 todo 應用程式。

## 安裝 React Redux

預設上，[React 綁定](https://github.com/gaearon/react-redux) 不包含在 Redux 中。你需要明確地安裝它：

```
npm install --save react-redux
```

## Container 和 Presentational Components

Redux 的 React 綁定擁抱了[分離 container 和 presentational components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) 的概念。

只讓應用程式的頂層 components (像是 route handlers) 意識到 Redux 是比較明智的。在它們下面的 Components 應該是 presentational 的並借由 props 接收所有的資料。

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Container Components</th>
            <th scope="col" style="text-align:left">Presentational Components</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">位置</th>
          <td>頂層、route handlers</td>
          <td>中間或分支末端的 components</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">意識到 Redux</th>
          <td>是</th>
          <td>否</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">取得資料</th>
          <td>訂閱 Redux state</td>
          <td>從 props 讀取資料</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">改變資料</th>
          <td>Dispatch Redux actions</td>
          <td>從 props 呼叫 callbacks</td>
        </tr>
    </tbody>
</table>

在這個 todo 應用程式中，我們將會只有一個 container component 在我們視圖階層的最上面。在更複雜的應用程式中，你可能會有幾個。當你可能會嵌套 container components 時，我們建議只要有可能就用傳遞 props 下去的方式。

## 設計 Component 階層

記得我們如何[設計 root state 物件的形狀](Reducers.md)的嗎？是時候來設計符合它的 UI 階層了。這不是個 Redux 特有的任務。[Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 是個解釋了這個過程的偉大教學。

我們的設計概要很簡單。我們想要顯示一個 todo 項目的清單。在點擊的時候，todo 項目被劃掉當作已完成。我們想要顯示一個使用者可以添加新 todo 的欄位。在 footer，我們想要顯示一個開關來切換 全部/只有已完成的/只有未完成的 todos。

我從這份概要中看到有下述 components (和它們的 props) 出現：

* **`AddTodo`** 是一個有一個按鈕的輸入欄位。
  - `onAddClick(text: string)` 是一個當按鈕被按下去時呼叫的 callback。
* **`TodoList`** 是一個顯示可見 todos 的清單。
  - `todos: Array` 是一個有著 `{ text, completed }` 形狀的 todo 項目的陣列。
  - `onTodoClick(index: number)` 是一個當 todo 被點擊時呼叫的 callback。
* **`Todo`** 是單一一個 todo 項目。
  - `text: string` 是要顯示的文字。
  - `completed: boolean` 是 todo 是否應該顯示為被劃掉。
  - `onClick()` 是一個當 todo 被點擊時呼叫的 callback。
* **`Footer`** 是一個我們讓使用者改變可見 todo 過濾條件的 component。
  - `filter: string` 是現在的的過濾條件：`'SHOW_ALL'`、`'SHOW_COMPLETED'` 或是 `'SHOW_ACTIVE'`。
  - `onFilterChange(nextFilter: string)`：當使用者選擇不同的過濾條件要呼叫的 Callback。

它們都是 presentational components。它們不知道資料從*哪裡*來，或是要*如何*改變它。它們只 render 給它們的東西。

如果你從 Redux 遷移到其他東西上，你將會可以讓這所有的 components 完全保持一樣。它們不依賴在 Redux 上。

讓我們來撰寫它們！我們還不需要思考有關綁定到 Redux 的部分。當你實驗的時候，你可以只是給它們假資料直到他們正常的 render 為止。

## Presentational Components

它們都是一般的 React components，所以我們不會停下來詳細介紹它們。直接開始：

#### `components/AddTodo.js`

```js
import React, { Component, PropTypes } from 'react'

export default class AddTodo extends Component {
  render() {
    return (
      <div>
        <input type='text' ref='input' />
        <button onClick={e => this.handleClick(e)}>
          Add
        </button>
      </div>
    )
  }

  handleClick(e) {
    const node = this.refs.input
    const text = node.value.trim()
    this.props.onAddClick(text)
    node.value = ''
  }
}

AddTodo.propTypes = {
  onAddClick: PropTypes.func.isRequired
}
```

#### `components/Todo.js`

```js
import React, { Component, PropTypes } from 'react'

export default class Todo extends Component {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none',
          cursor: this.props.completed ? 'default' : 'pointer'
        }}>
        {this.props.text}
      </li>
    )
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
}
```

#### `components/TodoList.js`

```js
import React, { Component, PropTypes } from 'react'
import Todo from './Todo'

export default class TodoList extends Component {
  render() {
    return (
      <ul>
        {this.props.todos.map((todo, index) =>
          <Todo {...todo}
                key={index}
                onClick={() => this.props.onTodoClick(index)} />
        )}
      </ul>
    )
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
}
```

#### `components/Footer.js`

```js
import React, { Component, PropTypes } from 'react'

export default class Footer extends Component {
  renderFilter(filter, name) {
    if (filter === this.props.filter) {
      return name
    }

    return (
      <a href='#' onClick={e => {
        e.preventDefault()
        this.props.onFilterChange(filter)
      }}>
        {name}
      </a>
    )
  }

  render() {
    return (
      <p>
        Show:
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    )
  }
}

Footer.propTypes = {
  onFilterChange: PropTypes.func.isRequired,
  filter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}
```

就這樣！我們可以藉由撰寫一個假的 `App` 來 render 它們，驗證它們正確的運作：

#### `containers/App.js`

```js
import React, { Component } from 'react'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

export default class App extends Component {
  render() {
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            console.log('add todo', text)
          } />
        <TodoList
          todos={
            [
              {
                text: 'Use Redux',
                completed: true
              },
              {
                text: 'Learn to connect it to React',
                completed: false
              }
            ]
          }
          onTodoClick={index =>
            console.log('todo clicked', index)
          } />
        <Footer
          filter='SHOW_ALL'
          onFilterChange={filter =>
            console.log('filter change', filter)
          } />
      </div>
    )
  }
}
```

這就是我在我 render `<App />` 時看到的:

<img src='http://i.imgur.com/lj4QTfD.png' width='40%'>

就它本身而言，不是非常有趣。讓我們來把它連結到 Redux！

## 連結 Redux

我們需要做兩個改變以把我們的 `App` component 連結到 Redux，並讓它 dispatch actions 還有從 Redux store 讀取 state。

首先，我們需要從我們前面安裝的 [`react-redux`](http://github.com/gaearon/react-redux) import `Provider`，並在 rendering 之前把 **root component 包在 `<Provider>` 中** 。

#### `index.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './containers/App'
import todoApp from './reducers'

let store = createStore(todoApp)

let rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

這使我們的 store 實體可以在下面的 components 中取得。(在內部，這是透過 React 的[「context」功能](http://facebook.github.io/react/docs/context.html)完成。)

接著，我們**把想要連結到 Redux 的 components 用來自 [`react-redux`](http://github.com/gaearon/react-redux) 的 `connect()` function 包起來**。請試著只對頂層 component、或是 route handlers 做這件事。雖然技術上你可以 `connect()` 你的應用程式中的任何 component 到 Redux store，請避免在太深的地方做這件事，因為這會讓資料流比較難追蹤。

**用 `connect()` 呼叫包覆的任何 component 都會接收一個 [`dispatch`](../api/Store.md#dispatch) function ，和來自全域 state 中任何它需要的 state 作為 prop。**在大部份的情況下，你只需要傳遞第一個參數給 `connect()`， 而它是一個我們稱作 **selector** 的 function。這個 function 取得全域 Redux store 的 state，然後回傳你需要的 props 給 component。在最簡單的案例中，你可以直接回傳給你的 `state`，不過你也可能希望先轉換它。

要用可組合的 selectors 來產生會自動記憶的高效能轉換，請查看 [reselect](https://github.com/faassen/reselect)。在這個範例中，我們不會使用它，不過它在比較大的應用程式中運作得很棒。

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

class App extends Component {
  render() {
    // 藉由 connect() 呼叫注入：
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    )
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}

function selectTodos(todos, filter) {
  switch (filter) {
    case VisibilityFilters.SHOW_ALL:
      return todos
    case VisibilityFilters.SHOW_COMPLETED:
      return todos.filter(todo => todo.completed)
    case VisibilityFilters.SHOW_ACTIVE:
      return todos.filter(todo => !todo.completed)
  }
}

// 我們想要從給定的全域 state 注入哪些 props？
// 附註：使用 https://github.com/faassen/reselect 可以獲得更好的效能。
function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  }
}

// 把 component 包起來以注入 dispatch 和 state 進去
export default connect(select)(App)
```

就這樣！這個小型的 todo 應用程式現在可以正確的運作了。

## 下一步

閱讀[這份教學的完整原始碼](ExampleTodoList.md)以更好地內化已獲得的知識。接著，直接前往[進階教學](../advanced/README.md)學習如何處理網路請求和 routing！
