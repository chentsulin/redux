# 計算衍生資料

[Reselect](https://github.com/faassen/reselect.git) 是一個簡單的 library 可以用來建立會自動記憶、可組合的 **selector** functions。可以用 Reselect 的 selectors 來有效率的從 Redux store 計算衍生資料。

### 使用 Memoized Selectors 的動機

讓我們再來看看 [Todos 清單的範例](../basics/UsageWithReact.md)：

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
    // 藉由 connect() 呼叫來注入：
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={this.props.visibleTodos}
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

function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  }
}

// 把 component 包起來以注入 dispatch 和 state 進去
export default connect(select)(App)
```

在上面的範例中，`select` 呼叫 `selectTodos` 以計算 `visibleTodos`。這運作得很好，不過有一個缺點：在每次 component 更新的時候都會計算 `visibleTodos`。如果 state tree 很大，或是計算的代價很大，在每次更新的時候重複計算可能會造成效能問題。Reselect 可以幫助我們避免這些不需要的重複計算。

### 建立一個 Memoized Selector

我們會想要把 `select` 換成一個 memoized selector，它會在 `state.todos` 或是 `state.visibilityFilter` 的值變更的時後重新計算 `visibleTodos`，但當變更發生在 state tree 其他 (不相關的) 的部分的時候則不會。

Reselect 提供一個 `createSelector` function 來建立 memoized selectors。`createSelector` 接收一個 input-selectors 陣列和一個轉換 function 當作它的參數。如果用一個會造成 input-selector 的值改變的方式去變動 Redux 的 state tree，selector 將會以 input-selectors 的值作為參數呼叫它的轉換 function 並回傳結果。如果 input-selectors 的值跟先前呼叫 selector 的時候一樣，它會回傳先前的計算結果而不會呼叫轉換 function。

讓我們來定義一個叫做 `visibleTodosSelector` 的 memoized selector 來取代 `select`：

#### `selectors/todoSelectors.js`

```js
import { createSelector } from 'reselect'
import { VisibilityFilters } from './actions'

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

const visibilityFilterSelector = (state) => state.visibilityFilter
const todosSelector = (state) => state.todos

export const visibleTodosSelector = createSelector(
  [ visibilityFilterSelector, todosSelector ],
  (visibilityFilter, todos) => {
    return {
      visibleTodos: selectTodos(todos, visibilityFilter),
      visibilityFilter
    }
  }
)
```

在上面的範例中，`visibilityFilterSelector` 和 `todosSelector` 就是 input-selectors。因為它們沒有轉換它們選擇的資料，所以被建立成普通的非 memoized selector functions。而另一方面，`visibleTodosSelector` 是一個 memoized selector。它接收 `visibilityFilterSelector` 和 `todosSelector` 作為 input-selectors，以及一個計算過濾後的 todos 清單的轉換 function。

### 組合 Selectors

一個 memoized selector 可以是另一個 memoized selector 的 input-selector。這裡 `visibleTodosSelector` 被用來當作 selector 的 input-selector，以藉由 keyword 進一步的過濾 todos：

```js
const keywordSelector = (state) => state.keyword

const keywordFilterSelector = createSelector(
  [ visibleTodosSelector, keywordSelector ],
  (visibleTodos, keyword) => visibleTodos.filter(
    todo => todo.indexOf(keyword) > -1
  )
)
```

### 把 Selector 連結到 Redux Store

如果你是使用 react-redux，你可以藉由 `connect` 把 memoized selector 連結到 Redux store：

#### `containers/App.js`

```js
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo, completeTodo, setVisibilityFilter } from '../actions'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'
import { visibleTodosSelector } from '../selectors/todoSelectors'

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
          todos={this.props.visibleTodos}
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

// 傳遞 selector 到 connect component
export default connect(visibleTodosSelector)(App)
```

