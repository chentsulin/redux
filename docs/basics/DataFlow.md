# 資料流

Redux 架構圍繞著**嚴格的單向資料流**。

這意味著應用程式中的全部資料都遵照一樣的生命週期模式，讓應用程式的邏輯更容易預測與了解。它也鼓勵資料正規化，這樣你就不會落得同份資料的多個獨立複製都互相不知道彼此的下場。

如果你仍然沒有被說服，請參閱[動機](../introduction/Motivation.md)和 [The Case for Flux](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6) 獲得一個令人信服的理由來贊成單向資料流。雖然 [Redux 不完全是 Flux](../introduction/PriorArt.md)，但是它共享一樣的關鍵優勢。

任何 Redux 應用程式中的資料生命週期都遵照這 4 個步驟：

1. **你呼叫** [`store.dispatch(action)`](../api/Store.md#dispatch)。

  [action](Actions.md) 是一個描述*發生什麼事*的一般物件。例如：

    ```js
    { type: 'LIKE_ARTICLE', articleId: 42 }
    { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Mary' } }
    { type: 'ADD_TODO', text: 'Read the Redux docs.' }
    ```

  把 action 想成是一個非常簡短的消息片段。「Mary liked article 42.」或是「Read the Redux docs.」被加到 todos 清單。

  你可以在應用程式的任何地方呼叫 [`store.dispatch(action)`](../api/Store.md#dispatch)，包括 components 和 XHR callbacks，或甚至在排程的 intervals。

2. **Redux store 呼叫你給它的 reducer function。**

  [store](Store.md) 會傳遞兩個參數到 [reducer](Reducers.md)，現在的 state tree 和 action。例如，在 todo 應用程式中，root reducer 可能會收到像這樣的東西：

    ```js
    // 現在的應用程式 state (todos 清單和挑選的過濾條件)
    let previousState = {
      visibleTodoFilter: 'SHOW_ALL',
      todos: [
        {
          text: 'Read the docs.',
          complete: false
        }
      ]
    }

    // 正要被執行的 action (添加一個 todo)
    let action = {
      type: 'ADD_TODO',
      text: 'Understand the flow.'
    }

    // 你的 reducer 會回傳應用程式的下一個 state
    let nextState = todoApp(previousState, action)
    ```

    請記得 reducer 只是個 pure function。它只*計算*下一個 state。它應該是完全可以預測的：用相同的 inputs 多次呼叫它應該產生相同的 outputs。它不應該執行任何有 side effects 的動作，像是 API 呼叫或是 router transitions。這些應該在 action 被 dispatched 之前發生。

3. **root reducer 可以把多個 reducers 的 output 合併成一個單一的 state tree。**

  要如何建構 root reducer 完全取決於你。Redux 附帶一個 [`combineReducers()`](../api/combineReducers.md) helper function，用於把 root reducer「拆分」成數個獨立 functions，個別管理 state tree 的一個分支。

  下面就是 [`combineReducers()`](../api/combineReducers.md) 的運作方式。比方說，你有兩個 reducers，一個針對 todos 清單，然後另一個針對現在被選擇的過濾設定：

    ```js
    function todos(state = [], action) {
      // Somehow calculate it...
      return nextState
    }

    function visibleTodoFilter(state = 'SHOW_ALL', action) {
      // Somehow calculate it...
      return nextState
    }

    let todoApp = combineReducers({
      todos,
      visibleTodoFilter
    })
    ```

  當你發出一個 action 時，`combineReducers` 回傳的 `todoApp` 將會兩個 reducers 都呼叫：

    ```js
    let nextTodos = todos(state.todos, action)
    let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action)
    ```

  它將會接著結合兩組結果成一個單一的 state tree：

    ```js
    return {
      todos: nextTodos,
      visibleTodoFilter: nextVisibleTodoFilter
    }
    ```

  儘管 [`combineReducers()`](../api/combineReducers.md) 是個方便的 helper utility，但你不一定要使用它； 請自由的撰寫你自己的 root reducer！

4. **Redux store 儲存 root reducer 回傳的完整 state tree。**

  這個新的 tree 現在是應用程式接下來的 state！每一個用 [`store.subscribe(listener)`](../api/Store.md#subscribe) 註冊的 listener 現在將會被呼叫；listeners 可以呼叫 [`store.getState()`](../api/Store.md#getState) 來取得現在的 state。

  現在，UI 可以被更新來反映新的 state。如果你使用像是 [React Redux](https://github.com/gaearon/react-redux) 之類的綁定，這就是 `component.setState(newState)` 被呼叫的地方。

## 下一步

現在你知道 Redux 如何運作了，讓我們來把[它連結到 React 應用程式](UsageWithReact.md)。

>##### 給進階使用者的附註
>如果你已經熟悉基礎概念並且先前已經完成了這個教學，請不要忘記查看[進階教學](../advanced/README.md)中的[非同步資料流](../advanced/AsyncFlow.md)，來學習 middleware 如何在 [async actions](../advanced/AsyncActions.md) 進入到 reducer 之前轉換它們。
