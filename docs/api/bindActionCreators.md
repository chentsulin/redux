# `bindActionCreators(actionCreators, dispatch)`

把一個每個值都是 [action creators](../Glossary.md#action-creator) 的物件轉換成另一個有同樣的 keys 的物件，不過每個 action creator 都被包進一個 [`dispatch`](Store.md#dispatch) 呼叫裡面，所以它們可以直接被呼叫。

通常你應該只需要直接在你的 [`Store`](Store.md) 實體上呼叫 [`dispatch`](Store.md#dispatch)。如果你結合 Redux 和 React 一起使用，[react-redux](https://github.com/gaearon/react-redux) 將會提供你 [`dispatch`](Store.md#dispatch) function，所以你也一樣可以直接地呼叫它。

`bindActionCreators` 唯一的使用情境是，當你希望傳遞一些 action creators 下去一個不知道 Redux 存在的 component，而你不希望把 [`dispatch`](Store.md#dispatch) 或是 Redux 的 store 傳遞給它。

為方便起見，你也可以傳遞單一一個 function 作為第一個參數，並在回傳中得到一個 function。

#### 參數

1. `actionCreators` (*Function* 或 *Object*)：一個 [action creator](../Glossary.md#action-creator)，或一個每個值都是 action creators 的物件。

2. `dispatch` (*Function*)：一個在 [`Store`](Store.md) 實體上可以取用的 [`dispatch`](Store.md#dispatch) function。

#### 回傳

(*Function* 或 *Object*)：一個模仿原始物件的物件，但是每個 function 都會立刻 dispatch 對應的 action creator 所回傳的 action。如果你傳遞一個 function 作為 `actionCreators`，回傳的值也會是單一一個 function。

#### 範例

#### `TodoActionCreators.js`

```js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
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

#### `SomeComponent.js`

```js
import { Component } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import * as TodoActionCreators from './TodoActionCreators'
console.log(TodoActionCreators)
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  componentDidMount() {
    // 藉由 react-redux 注入：
    let { dispatch } = this.props

    // 附註：這樣是無法運作的：
    // TodoActionCreators.addTodo('Use Redux')

    // 你只是呼叫一個會建立一個 action 的 function。
    // 你也必須 dispatch 這個 action！

    // 這將會正常運作：
    let action = TodoActionCreators.addTodo('Use Redux')
    dispatch(action)
  }

  render() {
    // 藉由 react-redux 注入：
    let { todos, dispatch } = this.props

    // 這裡是一個 bindActionCreators 極佳的使用情境：
    // 你希望 child component 完全不知道 Redux 的存在。

    let boundActionCreators = bindActionCreators(TodoActionCreators, dispatch)
    console.log(boundActionCreators)
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }

    return (
      <TodoList todos={todos}
                {...boundActionCreators} />
    )

    // 除了 bindActionCreators 之外的另一種選擇是
    // 把 dispatch function 傳遞下去，不過接著你的 child component
    // 需要 import action creators 並了解它們。

    // return <TodoList todos={todos} dispatch={dispatch} />
  }
}

export default connect(
  state => ({ todos: state.todos })
)(TodoListContainer)
```

#### 提示

* 你可能會問：為什麼我們不像傳統的 Flux 直接把 action creators 綁定到 store 實體上呢？問題是這在需要在伺服器 render 的 universal 應用程式上不會運作得很好。你很可能想要對每個請求擁有一個獨立的 store 實體，這讓你可以使用不同的資料來準備它們，但是在 action creators 定義時就綁定它們意味著你不得不對所有請求使用同一個 store 實體。

* 如果你使用 ES5，你可以把 `require('./TodoActionCreators')` 作為第一個參數傳遞到 `bindActionCreators` 來取代 `import * as` 語法。它唯一在意的事情是 `actionCreators` 參數的值是 functions。module 系統並不重要。
