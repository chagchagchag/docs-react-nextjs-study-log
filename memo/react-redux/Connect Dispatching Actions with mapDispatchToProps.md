# Connect: Dispatching Actions with `mapDispatchToProps`

원문 출처 : https://react-redux.js.org/using-react-redux/connect-mapdispatch

<br/>



`connect()` 에 전달된 두 번째 인수는 store에 action을 dispatch하는 데 사용되는 mapDispatchToProps입니다.

dispatch는 Redux store의 함수입니다. store.dispatch를 호출하여 action을 dispatch합니다. 이것이 상태 변경을 트리거하는 유일한 방법입니다.

React Redux를 사용하면 컴포넌트가 store에 직접 액세스하지 않고 connect가 이를 대신합니다. React Redux는 컴포넌트가 action을 dispatch하도록 하는 두 가지 방법을 제공합니다:

- 기본적으로 connected component 는 `props.dispatch`를 받고 자체적으로 action 을 dispatch 할 수 있습니다.
- `connect`는 호출될 때 dispatch하는 함수를 생성하고 해당 함수를 컴포넌트에 프로퍼티로 전달할 수 있는 `mapDispatchToProps`라는 인수를 받을 수 있습니다.

`mapDispatchToProps` 함수는 보통 줄여서 `mapDispatch`라고 부르지만, 실제 사용되는 변수 이름은 원하는 대로 지정할 수 있습니다.



## Approaches for Dispatching

### Default : `dispatch` as a Prop

connect()에 두 번째 인수를 지정하지 않으면 컴포넌트는 기본적으로 dispatch를 받습니다. <br/>

e.g.

```jsx
connect()(MyComponent)
// which is equivalent with
connect(null, null)(MyComponent)

// or
connect(mapStateToProps /** no second argument */)(MyComponent)
```

이런 식으로 컴포넌트를 연결하면 컴포넌트는 props.dispatch를 받습니다. 이를 사용하여 스토어에 action을 dispatch할 수 있습니다.

```jsx
function Counter({ count, dispatch }) {
  return (
    <div>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>reset</button>
    </div>
  )
}
```

<br/>



### Providing A `mapDispatchToProps` Parameter

mapDispatchToProps를 제공하면 **컴포넌트가 dispatch 해야 할 action 을 지정**할 수 있습니다. 이를 통해 <u>action dispatch 함수를 프로퍼티로 제공</u>할 수 있습니다. 따라서 `props.dispatch(() => increment())` 를 호출하는 대신 `props.increment()` 를 직접 호출할 수 있습니다. 이렇게 하는 데에는 몇 가지 이유가 있습니다.

<br/>



#### More Declarative

첫째, `dispatch` 로직을 함수로 캡슐화하면 구현이 더 선언적입니다. 동작을 dispatch 하고 Redux 스토어가 데이터 흐름을 처리하도록 하는 것은 동작이 무엇을 하는지가 아니라 동작을 구현하는 방법입니다.

버튼이 클릭될 때 action 을 dispatch하는 것이 좋은 예가 될 수 있습니다. 버튼을 직접 연결하는 것은 개념적으로 의미가 없으며, 버튼 참조를 dispatch하는 것도 마찬가지입니다.

```jsx
// button needs to be aware of "dispatch"
<button onClick={() => dispatch({ type: "SOMETHING" })} />

// button unaware of "dispatch",
<button onClick={doSomething} />
```

버튼이 클릭될 때 action을 dispatch하는 것이 좋은 예가 될 수 있습니다. 버튼을 직접 연결하는 것은 개념적으로 의미가 없으며, 버튼 참조를 dispatch하는 것도 마찬가지입니다. 따라서  **`mapDispatchToProps`**을 직접 정의하면 연결된 컴포넌트는 더 이상 dispatch를 받지 않습니다.



#### Pass Down Action Dispatching Logic to (Unconnected) Child Components

또한 action dispatch 함수를 자식(연결되지 않은 - likely unconnected) 컴포넌트에 전달할 수 있는 기능도 있습니다. 이렇게 하면 더 많은 컴포넌트가 action을 dispatch할 수 있으면서도 Redux를 "unaware" 상태로 유지할 수 있습니다.

```jsx
// pass down toggleTodo to child component
// making Todo able to dispatch the toggleTodo action
const TodoList = ({ todos, toggleTodo }) => (
  <div>
    {todos.map((todo) => (
      <Todo todo={todo} onClick={toggleTodo} />
    ))}
  </div>
)
```

이것이 바로 React Redux의 connect가 하는 일입니다. Redux 스토어와 대화하는 로직을 캡슐화하여 사용자가 걱정할 필요가 없도록 해줍니다. 그리고 이것이 바로 여러분이 구현에서 완전히 활용해야 하는 부분입니다.

<br/>



## Two forms of mapDispatchToProps

The `mapDispatchToProps` parameter can be of two forms. While the function form allows more customization, the object form is easy to use.

- **Function form**: Allows more customization, gains access to `dispatch` and optionally `ownProps`
- **Object shorthand form**: More declarative and easier to use

> ⭐ **Note:** We recommend using the object form of `mapDispatchToProps` unless you specifically need to customize dispatching behavior in some way.

<br/>



## Defining `mapDispatchToProps` As A Function

Defining `mapDispatchToProps` as a function gives you the most flexibility in customizing the functions your component receives, and how they dispatch actions. You gain access to `dispatch` and `ownProps`. You may use this chance to write customized functions to be called by your connected components.

### Arguments

- 1\. `dispatch`
- 2\. `ownProps` (optional)

**`dispatch`**

The `mapDispatchToProps` function will be called with `dispatch` as the first argument. You will normally make use of this by returning new functions that call `dispatch()` inside themselves, and either pass in a plain action object directly or pass in the result of an action creator.

```jsx
const mapDispatchToProps = (dispatch) => {
  return {
    // dispatching plain actions
    increment: () => dispatch({ type: 'INCREMENT' }),
    decrement: () => dispatch({ type: 'DECREMENT' }),
    reset: () => dispatch({ type: 'RESET' }),
  }
}
```

You will also likely want to forward arguments to your action creators:

```jsx
const mapDispatchToProps = (dispatch) => {
  return {
    // explicitly forwarding arguments
    onClick: (event) => dispatch(trackClick(event)),

    // implicitly forwarding arguments
    onReceiveImpressions: (...impressions) =>
      dispatch(trackImpressions(impressions)),
  }
}
```

**`ownProps` ( optional )**

If your `mapDispatchToProps` function is declared as taking two parameters, it will be called with `dispatch` as the first parameter and the `props` passed to the connected component as the second parameter, and will be re-invoked whenever the connected component receives new props.<br/>

(connected component 에 새로운 props 가 수신될때마다 재호출(re-invoked) 됩니다)<br/>

This means, instead of re-binding new `props` to action dispatchers upon component re-rendering, you may do so when your component's `props` change.

(즉, 컴포넌트를 re-rendering 할 때 action dispatcher 에 새로운 `props` 를 바인딩하는 대신 컴포넌트의 `props` 가 변경될 때 이를 수행할 수 있습니다.)

**Binds on component mount**

```jsx
render() {
  return <button onClick={() => this.props.toggleTodo(this.props.todoId)} />
}

const mapDispatchToProps = dispatch => {
  return {
    toggleTodo: todoId => dispatch(toggleTodo(todoId))
  }
}
```

**Binds on `props` change**

```jsx
render() {
  return <button onClick={() => this.props.toggleTodo()} />
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    toggleTodo: () => dispatch(toggleTodo(ownProps.todoId))
  }
}
```

<br/>



### Return

Your `mapDispatchToProps` function should return a plain object:

- Each field in the object will become a separate prop for your own component, and the value should normally be a function that dispatches an action when called.
  - object 의 각 field 는 component 에 대한 별도의 prop 이 되며, value 는 일반적으로 호출시에 action 을 dispatch 하는 함수여야 합니다.
- If you use action creators ( as oppose to plain object actions ) inside `dispatch`, it is a convention to simply name the field key the same name as the action creator:
  - `dispatch` 내에서 action creator (plain object action 이 아닌) 를 사용하는 경우 field key 를 action creator 와 같은 이름으로 지어주는 것이 Convention 입니다.

```jsx
const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })
const reset = () => ({ type: 'RESET' })

const mapDispatchToProps = (dispatch) => {
  return {
    // dispatching actions returned by action creators
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch(reset()),
  }
}
```

The return of the `mapDispatchToProps` function will be merged to your connected component as props. You may call them directly to dispatch its action.

- `mapDispatchToProps` 함수의 return 값은 당신의 connected component 에 props 로 merge 됩니다.  

```jsx
function Counter({ count, increment, decrement, reset }) {
  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={reset}>reset</button>
    </div>
  )
}
```

(Full code of the Counter example is [in this CodeSandbox](https://codesandbox.io/s/yv6kqo1yw9))

<br/>



### Defining the `mapDispatchToProps` Function with `bindActionCreators` 

Wrapping these functions by hand is tedious, so Redux provides a function to simplify that.

> `bindActionCreators` turns an object whose values are [action creators](https://redux.js.org/glossary#action-creator), into an object with the same keys, but with every action creator wrapped into a [`dispatch`](https://redux.js.org/api/store#dispatch) call so they may be invoked directly. See [Redux Docs on `bindActionCreators`](https://redux.js.org/api/bindactioncreators)

`bindActionCreators` accepts two parameters:

1. A **`function`** (an action creator) or an **`object`** (each field an action creator)
2. `dispatch`

The wrapper functions generated by `bindActionCreators` will automatically forward all of their arguments, so you don't need to do that by hand.

```jsx
import { bindActionCreators } from 'redux'

const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })
const reset = () => ({ type: 'RESET' })

// binding an action creator
// returns (...args) => dispatch(increment(...args))
const boundIncrement = bindActionCreators(increment, dispatch)

// binding an object full of action creators
const boundActionCreators = bindActionCreators(
  { increment, decrement, reset },
  dispatch,
)
// returns
// {
//   increment: (...args) => dispatch(increment(...args)),
//   decrement: (...args) => dispatch(decrement(...args)),
//   reset: (...args) => dispatch(reset(...args)),
// }
```

To use `bindActionCreators` in our `mapDispatchToProps` function:

```jsx
import { bindActionCreators } from 'redux'
// ...

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ increment, decrement, reset }, dispatch)
}

// component receives props.increment, props.decrement, props.reset
connect(null, mapDispatchToProps)(Counter)
```

<br/>



### Manually Injecting `dispatch`

If the `mapDispatchToProps` argument is supplied, **the component will no longer receive the default `dispatch`**. You may bring it back by adding it manually to the return of your `mapDispatchToProps`, although most of the time you shouldn’t need to do this:

```jsx
import { bindActionCreators } from 'redux'
// ...

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
    ...bindActionCreators({ increment, decrement, reset }, dispatch),
  }
}
```

<br/>



## Defining `mapDispatchToProps` As An Object

You’ve seen that the setup for dispatching Redux actions in a React component follows a very similar process: define an action creator, wrap it in another function that looks like `(…args) => dispatch(actionCreator(…args))`, and pass that wrapper function as a prop to your component.

Because this is so common, `connect` supports an “object shorthand” form for the `mapDispatchToProps` argument: if you pass an object full of action creators instead of a function, `connect` will automatically call `bindActionCreators` for you internally.

**We recommend always using the “object shorthand” form of `mapDispatchToProps`, unless you have a specific reason to customize the dispatching behavior.**

Note that:

- Each field of the `mapDispatchToProps` object is assumed to be an action creator
- Your component will no longer receive `dispatch` as a prop

```jsx
// React Redux does this for you automatically:
;(dispatch) => bindActionCreators(mapDispatchToProps, dispatch)
```



> action creator 를 사용한 전체 표현식을 사용해서 mapToDispatchProps 인자값을 넘기는 방식 대신 connect() API 는 객체의 단축 표현(object shorthand) 을 제공합니다. action creator 들로 가득찬 r객체를  function 대신 전달하면 `connect` 는 내부적으로 자동으로 `bindActionCreators` 를 호출합니다.



Therefore, our `mapDispatchToProps` can simply be:

```jsx
const mapDispatchToProps = {
  increment,
  decrement,
  reset,
}
```

<br/>



Since the actual name of the variable is up to you, you might want to give it a name like `actionCreators`, or even define the object inline in the call to `connect`:

```jsx
import { increment, decrement, reset } from './counterActions'

const actionCreators = {
  increment,
  decrement,
  reset,
}

export default connect(mapState, actionCreators)(Counter)

// or
export default connect(mapState, { increment, decrement, reset })(Counter)
```

<br/>



## Common Problems

### Why is my component not receiving `dispatch`?

Also known as

```jsx
TypeError: this.props.dispatch is not a function
```



This is a common error that happens when you try to call `this.props.dispatch` , but `dispatch` is not injected to your component.

`dispatch` is injected to your component *only* when:

**1. You do not provide `mapDispatchToProps`**

The default `mapDispatchToProps` is simply `dispatch => ({ dispatch })`. If you do not provide `mapDispatchToProps`, `dispatch` will be provided as mentioned above.

In another words, if you do:

```jsx
// component receives `dispatch`
connect(mapStateToProps /** no second argument*/)(Component)
```

<br/>



**2. Your customized `mapDispatchToProps` function return specifically contains `dispatch`**

You may bring back `dispatch` by providing your customized `mapDispatchToProps` function:

```jsx
const mapDispatchToProps = (dispatch) => {
  return {
    increment: () => dispatch(increment()),
    decrement: () => dispatch(decrement()),
    reset: () => dispatch(reset()),
    dispatch,
  }
}
```

<br/>



Or alternatively, with `bindActionCreators`:

```jsx
import { bindActionCreators } from 'redux'

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
    ...bindActionCreators({ increment, decrement, reset }, dispatch),
  }
}
```



See [this error in action in Redux’s GitHub issue #255](https://github.com/reduxjs/react-redux/issues/255).

There are discussions regarding whether to provide `dispatch` to your components when you specify `mapDispatchToProps` ( [Dan Abramov’s response to #255](https://github.com/reduxjs/react-redux/issues/255#issuecomment-172089874) ). You may read them for further understanding of the current implementation intention.



### Can I `mapDispatchToProps` without `mapStateToProps` in Redux?

Yes. You can skip the first parameter by passing `undefined` or `null`. Your component will not subscribe to the store, and will still receive the dispatch props defined by `mapDispatchToProps`.

```jsx
connect(null, mapDispatchToProps)(MyComponent)
```

<br/>



### Can I call `store.dispatch`?

It's an anti-pattern to interact with the store directly in a React component, whether it's an explicit import of the store or accessing it via context (see the [Redux FAQ entry on store setup](https://redux.js.org/faq/storesetup#can-or-should-i-create-multiple-stores-can-i-import-my-store-directly-and-use-it-in-components-myself) for more details). Let React Redux’s `connect` handle the access to the store, and use the `dispatch` it passes to the props to dispatch actions.<br/>



## Links and References

**Tutorials**

- [You Might Not Need the `mapDispatchToProps` Function](https://daveceddia.com/redux-mapdispatchtoprops-object-form/)

**Related Docs**

- [Redux Doc on `bindActionCreators`](https://redux.js.org/api/bindactioncreators)

**Q&A**

- [How to get simple dispatch from `this.props` using connect with Redux?](https://stackoverflow.com/questions/34458261/how-to-get-simple-dispatch-from-this-props-using-connect-w-redux)
- [`this.props.dispatch` is `undefined` if using `mapDispatchToProps`](https://github.com/reduxjs/react-redux/issues/255)
- [Do not call `store.dispatch`, call `this.props.dispatch` injected by `connect` instead](https://github.com/reduxjs/redux/issues/916)
- [Can I `mapDispatchToProps` without `mapStateToProps` in Redux?](https://stackoverflow.com/questions/47657365/can-i-mapdispatchtoprops-without-mapstatetoprops-in-redux)
- [Redux Doc FAQ: React Redux](https://redux.js.org/faq/reactredux)