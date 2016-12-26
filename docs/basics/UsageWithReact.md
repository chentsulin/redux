# 搭配 React 運用

在一開始，我們必須強調，Redux 跟 React 並沒有關係。你可以用 React、Angular、Ember、jQuery 或甚至原生 JavaScript 來撰寫 Redux 應用程式。

不過，Redux 與像是 [React](http://facebook.github.io/react/) 和 [Deku](https://github.com/dekujs/deku) 之類的框架一起運作的特別好，因為它們讓你把 UI 描述成一個 state 的 function，而 Redux 對應 action 來發出 state 更新。

我們將會使用 React 來建置我們的簡易 todo 應用程式。

## 安裝 React Redux

預設上，[React 綁定](https://github.com/reactjs/react-redux) 不包含在 Redux 中。你需要明確地安裝它：

```
npm install --save react-redux
```

如果你沒有使用 npm，你可以從 npmcdn 取得最新的 UMD build (不論是 [development](https://unpkg.com/react-redux@latest/dist/react-redux.js) 或是 [production](https://unpkg.com/react-redux@latest/dist/react-redux.min.jss) 的 build)。如果你把它經由一個 `<script>` 標籤加入你的頁面，UMD build 就會 export 一個名為 `window.ReactRedux` 的全域變數。

## Presentational 和 Container Component

Redux 的 React 綁定擁抱了**分離 presentational 和 container component** 的概念。如果你還不熟悉這些詞彙，[先閱讀這些文章](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)，然後再回來這裡。這些文章很重要所以值得我們花時間！

看完這些文章了嗎？來說一下它們的差異：

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Presentational Components</th>
            <th scope="col" style="text-align:left">Container Components</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">用途</th>
          <td>怎麼看事情（markup, styles)</td>
          <td>怎麼做事情 (抓資料, 更新state)</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">意識到 Redux</th>
          <td>否</th>
          <td>是</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">取得資料</th>
          <td>從 props 讀取資料</td>
          <td>訂閱 Redux state</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">改變資料</th>
          <td>從 props 呼叫 callback</td>
          <td>Dispatch Redux action</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">從哪被寫入</th>
          <td>經由手動</td>
          <td>通常由 React Redux 產生</td>
        </tr>
    </tbody>
</table>

我們大多數是寫 presentational component，但我們將需要產生一些 container component 來連結 Redux store。

雖然技術上來說你可以使用 [`store.subscribe()`](../api/Store.md#subscribe) 來手動寫 container component，但我們不建議你這樣做，因為 React Redux 有做許多手寫難以達成的效能優化。因此，我們以下將使用  React Redux 提供的 [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) function 產生 container component 而非手動寫成。

## 設計 Component 階層

記得我們如何[設計 root state 物件的形狀](Reducers.md)的嗎？是時候來設計符合它的 UI 階層了。這不是個 Redux 特有的任務。[Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 是個解釋了這個過程的偉大教學。

我們的設計概要很簡單。我們想要顯示一個 todo 項目的清單。在點擊的時候，todo 項目被劃掉當作已完成。我們想要顯示一個使用者可以添加新 todo 的欄位。在 footer，我們想要顯示一個開關來切換 全部，或只有已完成的，或只有未完成的 todos。

### Presentational Component

我從這份概要中看到有下述 presentational component 和它們的 props 出現：

* **`TodoList`** 是一個顯示可見 todos 的清單。
  - `todos: Array` 是一個有著 `{ id, text, completed }` 形狀的 todo 項目的陣列。
  - `onTodoClick(id: number)` 是一個當 todo 被點擊時呼叫的 callback。
* **`Todo`** 是單一一個 todo 項目。
  - `text: string` 是要顯示的文字。
  - `completed: boolean` 是 todo 是否應該顯示為被劃掉。
  - `onClick()` 是一個當 todo 被點擊時呼叫的 callback。
* **`Link`** 是一個有 callback 的 link。
  - `onClick()` 是一個當 link 被點擊時呼叫的 callback。
* **`Footer`** 是我們讓使用者改變現在可見 todos 的位置。
* **`App`** 是 render 所有事物的 root component。

它們描述了*樣貌*卻不知道資料從*哪裡*來，或是要*如何*改變它。它們只 render 給它們的東西。如果你從 Redux 遷移到其他東西上，你將會可以讓這所有的 component 完全保持一樣。它們不依賴在 Redux 上。

### Container Component

我們也需要一些 container component 來連結 presentational component 至 Redux。舉例來說，presentational `TodoList` component 需要一個像是 `VisibleTodoList` 的 container 來訂閱 Redux store 並知道如何應用現在的顯示篩選器。為了改變顯示篩選器，我們將提供一個 `FilterLink` container component 來 render 一個在點擊後會 dispatch 一個適當 action 的 `Link`：

* **`VisibleTodoList`** 根據目前的顯示篩選器來篩選 todos 並 render 一個 `TodoList`。
* **`FilterLink`** 拿到目前的顯示篩選器並 render 一個 `Link`。
  - `filter: string` 呈現顯示篩選器。

### 其它 Component

有時候有些 component 很難分辨是 presentational component 還是 container component。像是有些 form 和 function 會如同這個微型 component 一樣確實地耦合在一起：

* **`AddTodo`** 是一個有 "Add" 按鈕的輸入欄位

技術上來說我們可以把它拆分為兩個 component，但在這個階段也許還太早了。混合 presentation 和 logic 在一個非常小的 component 中是可接受的。當它日漸增大時，要如何拆分它會更加明顯，所以我們將在這保持著混合。

## 實作 Component

動手寫 component 吧！我們從 presentational component 開始，所以還不需要想著怎麼綁定 Redux。

### Presentational Component

這些都是常態的 React component，所以我們不會詳細檢視它們。除非我們需要使用區域 state 或是 lifecycle method，不然我們將寫 functional stateless component。這不代表 presentational component *必須*
是 function－這只是簡單的定義方式。假如當你需要增加區域 state，或 lifecycle method，或效能優化，你可以將它們轉成 class。

#### `components/Todo.js`

```js
import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

#### `components/TodoList.js`

```js
import React, { PropTypes } from 'react'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

#### `components/Link.js`

```js
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

#### `components/Footer.js`

```js
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default Footer
```

#### `components/App.js`

```js
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
)

export default App
```

### Container Component

現在是時候建立些 container 使得那些 presentational component 接到 Redux 了。技術上來說，一個 container component 就只是個使用 [`store.subscribe()`](../api/Store.md#subscribe) 來讀取一部份 Redux state tree 並提供 props 讓 presentational component 來 render 用的 React component。你可以手寫一個 container component，但 React Redux 囊括了許多有用的優化，所以我們建議從 React Redux library 使用 [`connect()`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) function 來產生 container component，它提供了許多有用的最佳化來避免不必要的重新 render。（其中一個結果就是，你不必擔心關於 [React 的效能建議](https://facebook.github.io/react/docs/advanced-performance.html)而自己實作 `shouldComponentUpdate`。）

要使用 `connect()`，你需要定義一個名為 `mapStateToProps` 的特別 function，它述說將如何轉換目前 Redux store state 成為你想要傳到正在包裝的 presentational component 的 props。舉個例子，`VisibleTodoList` 需要計算 `todos` 來傳給 `TodoList`，所以我們定義一個根據 `state.visibilityFilter` 來篩選 `state.todos` 的 function，並在它的 `mapStateToProps` 中使用：

```js
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

除了讀取 state，container component 可以 dispatch actions。以類似的方式，你可以定義一個名為 `mapDispatchToProps()` 的 function，它接收 [`dispatch()`](../api/Store.md#dispatch) method 並回傳你想要注入 presentational component 的 callback props。舉個例子，我們想要 `VisibleTodoList` 注入一個名為 `onTodoClick` 的 prop 到 `TodoList` component，並且我們想要 `onTodoClick` 來 dispatch 一個 `TOGGLE_TODO` action：

```js
const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}
```

最後，我們呼叫 `connect()` 來建立 `VisibleTodoList` 並傳入這兩個 function：

```js
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

這就是 React Redux API 的基礎，除此之外另有一些方便用法和厲害的選項，所以我們鼓勵你仔細查看 [它的文件](https://github.com/reactjs/react-redux)。萬一你擔心 `mapStateToProps` 會太常建立新的 object，那你可能會想要了解[計算衍生資料](../recipes/ComputingDerivedData.md) 和 [reselect](https://github.com/reactjs/reselect)。

在下面找到 container component 剩餘定義的部分：

#### `containers/FilterLink.js`

```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

#### `containers/VisibleTodoList.js`

```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

### 其它 Components

#### `containers/AddTodo.js`

```js
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form onSubmit={e => {
        e.preventDefault()
        if (!input.value.trim()) {
          return
        }
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        <input ref={node => {
          input = node
        }} />
        <button type="submit">
          Add Todo
        </button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

## 傳遞 Store

所有 container component 都需要接到 Redux store，所以它們才能 subscribe 它。其中一種方式是把它當作 props 來傳遞給每個 container component。然而當你甚至必須串 `store` 至 presentational component 只因為它們在 component tree 的深處有 render 一個 container 時，這將變得單調乏味。

我們推薦的方式是使用一個名為 [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) 的特別 React Redux component 來 [神奇地](https://facebook.github.io/react/docs/context.html) 使所有應用程式中的 container component 可取得 store，而非明確地傳遞它進去。你只需要在你 render 的 root component 中使用一次即可：

#### `index.js`

```js
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

## 下一步

閱讀[這份教學的完整原始碼](ExampleTodoList.md)以更好地內化已獲得的知識。接著，直接前往[進階教學](../advanced/README.md)學習如何處理網路請求和 routing！
