## react-redux Todo App (1)

> 참고 : plain redux 가 아닌 react-redux 에 대해 개념을 정리합니다.

공식 문서에서 제공하는 todo app 예제를 통해 개념을 정리해봅니다.<br/>

현재 버전의 공식 문서에서는 비교적 복잡한 버전의 Todo App 을 사용하는데, 복잡한 버전에 대해서는 추후 다뤄보기로 하고, 이번 문서에서는 다소 간단한 버전의 Todo App 을 통해 redux 의 개념을 살펴봅니다.<br/>

예제의 링크는 [codesandbox](https://codesandbox.io/s/0vm2w0k9r0) 에 있습니다.<br/>



## 참고자료

> [Tutorial : Connect API](https://react-redux.js.org/tutorials/connect)

<br/>



**Jump to**

- 🤞 [Just show me the code](https://codesandbox.io/s/9on71rvnyo) 
- 👆 [Providing the store](https://react-redux.js.org/tutorials/connect#providing-the-store)
- ✌️ [Connecting the Component](https://react-redux.js.org/tutorials/connect#connecting-the-components)

<br/>



**API Reference**

- [connect()](https://react-redux.js.org/api/connect)

<br/>



## Redux

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*GUUEnrvJJTkKSx-XXdvW_w.png)

> 출처 : [https://medium.com/globant/react-state-management-b0c81e0cbbf3](https://medium.com/globant/react-state-management-b0c81e0cbbf3)

<br/>



## index.js - store 세팅 (App, Provider, store, rootReducer)

App 컴포넌트는 Provider 객체로 감싸서 ReacDom.render 를 통해 최상단에서 렌더링될 수 있도록 설정하고 있습니다.

```jsx
import React from "react";
import ReactDOM from "react-dom";
import App from "./components/App";
import { Provider } from "react-redux";
import { createStore } from "redux";
import rootReducer from "./reducers"; // (1)

const store = createStore(rootReducer); // (2)

const styles = {
  fontFamily: "sans-serif",
  textAlign: "center"
};

ReactDOM.render(
  // (3)
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```



(1) : rootReducer

- reducers/index.js 파일에서 combineReducer() 로 생성해서 export 한 Object 타입의 객체를 rootReducer 라는 이름으로 import 해서 얻어옵니다.



(2) : store

- Redux 의 createStore(rootReducer)로 얻어낸 store 객체를 생성합니다.



(3) : Provider

- Provider 의 store props 에 store 객체를 바인딩해줍니다.
- Provider 로 App 컴포넌트를 감싸서 store 를 전역으로 사용할 수 있도록 해줍니다.



![](./img/react-redux-todoapp/index-js.png)

<br/>



## src/reducers/index.js - rootReducer

src/reducers/index.js 내의 rootReducer 를 선언하는 코드는 아래와 같습니다.<br/>

**src/reducers/index.js**

```jsx
import { combineReducers } from "redux";
import todos from "./todo.reducers";
import filterTodo from "./filter.reducers";

export default combineReducers({
  // (1)
  todos, 
  filterTodo
});
```

<br/>



(1) 

- todos : src/reducers/todo.reducers.js 에서 불러온 reducer 입니다.
- filterTodo : src/reducers/filter.reducers.js 에서 불러온 reducer 입니다.

<br/>



참고로 todo.reducers.js 파일의 코드는 아래와 같습니다. todo.reducers.js, filter.reducers.js 는 뒤에서 자세히 정리합니다. 

```js
const todos = (state = [], action) => {
  switch (action.type) {
    case "ADD_TODO":
      return [
        ...state,
        {
          id: action.id,
          complete: action.complete,
          text: action.text
        }
      ];
    case "TOGGLE_TODO":
      return state.map(
        todo =>
          todo.id === action.id ? { ...todo, complete: !todo.complete } : todo
      );
    default:
      return state;
  }
};
export default todos;
```

<br/>



**src/reducers/index.js** 에서 combineReducers({...}) 으로 생성한 후 export 한 reducer 객체는 index.js 에서 아래와 같이 import 합니다(1). 그리고 이 reducer를 store 의 인자로 넘겨주어서 store를 생성하며(2), 최종적으로는 Provider 객체에 store 객체를 바인딩하는 것으로 App 객체의 선언을 마무리(3)합니다. index.js 의 코드는 이전 섹션에서 살펴보았습니다.

```jsx
// ...
import rootReducer from "./reducers"; // (1)

// ...
const store = createStore(rootReducer); // (2)

// ...
ReactDOM.render(
  // (3)
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

<br/>



## App.js - 컴포넌트 배치

컴포넌트의 구조는 아래와 같이 구성했습니다.<br/>

**src/components/App.js**

```jsx
import React from "react";
import AddTodo from "../containers/AddTodo";
import TodoList from "../containers/TodoList";
import Footer from "../containers/Footer";

const App = () => (
  <div>
    <AddTodo />
    <TodoList />
    <Footer />
  </div>
);

export default App;
```

<br/>



위 코드에서 배치한 각 컴포넌트는 브라우저 상에서는 아래와 같이 표현됩니다.

![](./img/react-redux-todoapp/app-js.png)

<br/>



## src/containers/AddTodo.js - AddTodo 컴포넌트

```jsx
import React from "react";
import { addTodo } from "../actions/todo.actions";
import { connect } from "react-redux";

const AddTodo = ({ dispatch }) => { // (1)
  let input;
  let onClick = (e) => {
    if (input.value.trim() != "") {
      dispatch(addTodo(input.value.trim()));
    }
  };
  return (
    <React.Fragment>
      <input type="text" ref={node => (input = node)} />
      <button type="submit" onClick={onClick}>
        Add Todo
      </button>
    </React.Fragment>
  );
};

export default connect()(AddTodo);
```

<br/>



(1) 

- AddTodo 컴포넌트에 props 로 인자를 받고 있는 `dispatch` 함수는 어디서 불러오는지 궁금해질 수 있는데 
- 





