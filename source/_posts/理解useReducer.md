---
title: 理解useReducer
date: 2020-05-20 10:33:51
tags: react
---
useReducer 是一个用于状态管理的 hook api
useReducer(reducer, initialState) 接受2个参数，分别为 reducer 函数 和 初始状态

### 简单state
源码：https://codesandbox.io/s/hooks-usereducer-qobdy?file=/src/CountOne.js
App.js

```javascript
import React from "react";
import "./styles.css";
import CountOne from "./CountOne";
import CountTwo from "./CountTwo";

export default function App() {
  return (
    <div className="App">
      <CountOne />
      <hr />
      <CountTwo />
    </div>
  );
}
```
CountOne:

```javascript
import React, { useReducer } from "react";

//初始化状态
const initState = 0;
const reducer = (state, action) => {
  switch (action) {
    case "increment":
      return state + 1;
    case "decrement":
      return state - 1;
    case "reset":
      return initState;
    default:
      return state;
  }
};

export default function CountOne() {
  const [count, dispatch] = useReducer(reducer, initState);
  return (
    <div>
      <div>Count - {count}</div>
      <button onClick={() => dispatch("increment")}>Increment</button>
      <button onClick={() => dispatch("decrement")}>Decrement</button>
      <button onClick={() => dispatch("reset")}>Reset</button>
    </div>
  );
}
```
### 复杂state
源码：https://codesandbox.io/s/hooks-usereducer-qobdy?file=/src/CountTwo.js

```javascript
import React, { useReducer } from "react";

//初始化状态
const initState = {
  first: 0
};
const reducer = (state = { first: 0 }, action = { value: 0 }) => {
  switch (action.type) {
    case "increment":
      return { first: state.first + action.value };
    case "decrement":
      return { first: state.first - action.value };
    case "reset":
      return initState;
    default:
      return state;
  }
};

export default function CountTwo() {
  const [count, dispatch] = useReducer(reducer, initState);
  return (
    <div>
      <div>Count - {count.first}</div>
      <button onClick={() => dispatch({ type: "increment", value: 1 })}>
        Increment
      </button>
      <button onClick={() => dispatch({ type: "decrement", value: 1 })}>
        Decrement
      </button>
      <button onClick={() => dispatch({ type: "reset", value: 0 })}>
        Reset
      </button>
    </div>
  );
}
```