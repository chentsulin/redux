# Reducer

[Action](./Actions.md) 描述實際*發生一些事*，不過並不指定應用程式的 state 要如何去應對改變。這是 reducer 的工作。

## 設計 State 的形狀

在 Redux 中，所有的應用程式 state 被儲存為一個單一物件。在撰寫任何程式碼之前先思考它的形狀是個好主意。如何把你的應用程式的 state 描述成一個最簡單形式的物件？

以我們的 todo 應用程式來說，我們想要儲存兩個不同的東西：

* 現在被選擇的可見度過濾條件；
* 實際的 todos 清單。

你常常會發現需要儲存一些資料，以及一些 UI 狀態在 state tree 中。這沒問題，不過請盡量讓資料和 UI 狀態分離。

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

>##### 關於 Relationship 的附註

>在更複雜的應用程式中，你會希望不同的實體可以互相參考到其他實體。我們建議你盡量保持 state 正規化，不要有任何的巢狀。讓儲存在物件的每個實體都用一個 ID 作為 key，並且使用 IDs 來從其他實體或清單參考它。把應用程式的 state 想成是一個資料庫。這個方法詳細的描述在 [normalizr 的](https://github.com/gaearon/normalizr) 文件中。例如，在一個真實的應用程式中，同時把 `todosById: { id -> todo }` 和 `todos: array<id>` 保存在 state 裡面會是比較好的方式，不過我們會盡量讓範例保持簡單。

## 處理 Action

現在我們已經決定 state 物件要長什麼樣子了，我們準備幫它撰寫一個 reducer。reducer 是一個 pure function，它接收先前的 state 和一個 action，然後回傳下一個 state。

```js
(previousState, action) => newState
```

它被稱作 reducer，是因為它是你傳遞到 [`Array.prototype.reduce(reducer, ?initialValue)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 的 function 類型。讓 reducer 保持 pure 非常重要。你**永遠不**應該在 reducer 裡面做這些事：

* 改變它的參數；
* 執行有 side effect 的動作，像是呼叫 API 和 routing 轉換。
* 呼叫不是 pure 的 function，像是 `Date.now()` 或是 `Math.random()`。

我們將會在[進階教學](../advanced/README.md)中探索如何執行有 side effects 的動作。現在，只要記得 reducer 必須是 pure 的就好。**給定一樣的參數，他必須一樣計算下一個 state 並回傳它。沒有驚喜。沒有 side effect。沒有 API 呼叫。沒有變更。只是一個計算。**

避開這些，讓我們開始來撰寫我們的 reducer，藉由漸漸的教它認得我們之前定義的 [actions](Actions.md)。

我們會從指定初始的 state 開始。第一次的時候，Redux 會用一個 `undefined` state 來呼叫我們的 reducer。這是我們可以回傳應用程式初始 state 的機會：

```js
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // 在現在，不要處理任何 action
  // 而只是回傳給我們的 state。
  return state
}
```

一個絕妙的技巧是使用 [ES6 參數預設值語法](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters)來用更簡潔的方式撰寫這個：

```js
function todoApp(state = initialState, action) {
  // 現在，不要處理任何 action
  // 而只是回傳給我們的 state。
  return state
}
```

現在讓我們來處理 `SET_VISIBILITY_FILTER`。它所需要做的是改變 state 中的 `visibilityFilter`。簡單的：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

注意這幾點：

1. **我們不改變 `state`。** 我們用 [`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 複製一份。`Object.assign(state, { visibilityFilter: action.filter })` 也是錯的：它會改變第一個參數。你**必須**提供一個空物件作為第一個參數。你也可以啟用 [object spread 運算子提案](../recipes/UsingObjectSpreadOperator.md)，就可以寫 `{ ...state, ...newState }` 作為取代。

2. **我們在 `default` case 回傳先前的 `state`。**針對任何未知的 action 回傳先前的 `state` 非常重要。

>##### 關於 `Object.assign` 的附註

>[`Object.assign()`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 是 ES6 的一部分，不過大部份瀏覽器尚未實作。你會需要使用一個 polyfill，[Babel plugin](https://www.npmjs.com/package/babel-plugin-transform-object-assign) 或是從其他 library 來的 helper 像是 [`_.assign()`](https://lodash.com/docs#assign)。

>##### 關於 `switch` 和 Boilerplate 的附註

>`switch` 語句*不*算真實的 boilerplate。真實的 Flux boilerplate 是概念性的：需要發送更新、需要註冊 Store 到 Dispatcher、Store 需要是一個物件 (並在你想要一個 universal 應用程式的時候出現併發症)。Redux 藉由使用 pure reducer 取代 event emitter 解決了這些問題。

>不幸的事，許多人仍然依照它是不是在文件中使用 `switch` 語句來選擇框架。如果你不喜歡 `switch`，你可以使用一個接收 handler map 的自訂 `createReducer` function，如[「減少 boilerplate」](../recipes/ReducingBoilerplate.md#reducers)中所示。

## 處理更多 Action

我們有兩個以上的 action 要處理！就像我們做的 `SET_VISIBILITY_FILTER`，我們將 import `ADD_TODO` 和 `TOGGLE_TODO` action 並讓我們來擴充 reducer 以處理 `ADD_TODO`。

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

就像之前一樣，我們從來不直接寫入 `state` 或是他的屬性，取而代之我們回傳一個新的物件。新的 `todos` 等同於舊的 `todos` 在尾端串接一個新項目。新的 todo 項目是使用從 action 來的資料建構而成。

最後，`TOGGLE_TODO` handler 的實作應該完全不意外：

```js
case TOGGLE_TODO:
  return Object.assign({}, state, {
    todos: state.todos.map((todo, index) => {
      if (index === action.index) {
        return Object.assign({}, todo, {
          completed: !todo.completed
        })
      }
      return todo
    })
  })
```

因為我們想要更新陣列中的一個特定項目而不採用改變的方式，我們必須建立一個有相同元素的新的陣列，除了那個特定元素的索引。如果你發覺自己時常撰寫這樣的操作，使用像是 [react-addons-update](https://facebook.github.io/react/docs/update.html)、[updeep](https://github.com/substantial/updeep) 之類的 helper、或甚至像是 [Immutable](http://facebook.github.io/immutable-js/) 之類有原生支援深層更新的 library 是個好主意。要記住永遠不要 assign 到任何 `state` 裡面的東西，除非你先 clone 它。

## 拆分 Reducers

這是我們到目前為止的程式碼。這相當的詳細：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if(index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

有方法讓它更容易理解嗎？`todos` 和 `visibilityFilter` 的更新似乎是完全獨立的。有時候 state 屬性會互相依賴而需要更多的思考，不過在我們這個案例中，我們可以簡單的把 `todos` 的更新拆分成一個獨立的 function：

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

請注意 `todos` 也是接收 `state`—不過它是個陣列！現在只給 `todoApp` state 的一部分去管理，而 `todos` 也只知道如何更新那一部分。**這被稱為 *reducer composition*，它是建置 Redux 應用程式的基本模式。**

讓我們來探索更多有關 reducer composition 的部分。我們也能抽出一個只管理 `visibilityFilter` 的 reducer 嗎？我們可以：

以下我們使用 import，讓我們使用 [ES6 物件解構](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)來宣告 `SHOW_ALL`：
```js
const { SHOW_ALL } = VisibilityFilters;
```

接著：
```js
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}
```

現在我們可以把主要的 reducer 改寫成一個 function，它呼叫數個 reducer 來分別管理 state 的一部分，並把它們合併成單一一個物件。它也不再需要知道完整的初始 state 了。子 reducers 們在它們一開始被給予 `undefined 的時候只要回傳它們的初始 state 就足夠了。

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

**請記住這些 reducer 每一個都管理它所擁有的全域 state 一部分。每個 reducer 的 `state` 參數都不一樣，並對應到它管理的部分 state。**

這已經看起來蠻棒的了！隨著應用程式大小增長，我們可以把 reducers 拆分成個別的檔案並讓它們完全獨立並管理不同的資料領域。

最後，Redux 提供一個 utility 叫做 [`combineReducers()`](../api/combineReducers.md)，它做了與上面 `todoApp` 做的事情一樣的 boilerplate 邏輯。有它的幫助，我們可以像這樣改寫 `todoApp`：

```js
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

請注意這完全等同於：

```js
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

你也可以給它們不同的 key、或呼叫不同的 function。這兩個撰寫組合的 reducer 的方法完全相等：

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})
```

```js
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

[`combineReducers()`](../api/combineReducers.md) 做的就是產生一個 function，它**以依照它們的 key 所選擇的 state 部分來**呼叫你的 reducers，並再次把它們的結果合併成單一一個物件。[它不是魔法。](https://github.com/reactjs/redux/issues/428#issuecomment-129223274)

>##### 給理解 ES6 的使用者們的附註

>因為 `combineReducers` 預期會接收到一個物件，我們可以把所有底層 reducer 放進個別的檔案中，`export` 每一個 reducer function，然後使用 `import * as reducers` 取得一個以它們的名字作為 keys 的物件：

>```js
>import { combineReducers } from 'redux'
>import * as reducers from './reducers'
>
>const todoApp = combineReducers(reducers)
>```
>
>因為 `import *` 還是一個新的語法，我們今後不會在文件中使用它以避免[困惑](https://github.com/reactjs/redux/issues/428#issuecomment-129223274)，不過你可能會在一些社群的範例中遇到它。

## 原始碼

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import { ADD_TODO, TOGGLE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## 下一步

接下來，我們將探索如何[建立 Redux store](Store.md)，它會掌管 state 並在你 dispatch 一個 action 時幫忙呼叫 reducer。
