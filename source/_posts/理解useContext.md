---
title: 理解useContext
date: 2020-05-19 11:27:33
tags: react
---
## 什么是 Context
考虑这样一种场景，如果组件树结构如下，现在想从根节点传递一个 userName 的属性到叶子节点 A D F，通过 props 的方式传递，会不可避免的传递通过 B C E，即使这些组件也没有使用这个 userName 属性。
如果这样的嵌套树形结构有5层或10层，那么将是灾难式的开发维护体验。如果能不经过中间的节点直接到达需要的地方就可以避免这种问题，这时 Context 就是来解决这个问题的。

## 使用 Context
源码地址：https://codesandbox.io/s/react-context-w56oe
App:

```javascript
import React from "react";
import "./styles.css";
import ComponentA from "./ComponentA";

export const UserContext = React.createContext("");

export default function App() {
  return (
    <div className="App">
      <UserContext.Provider value={"111"}>
        <ComponentA />
      </UserContext.Provider>
    </div>
  );
}
```
ComponentA:

```javascript
import React from "react";

import ComponentB from "./ComponentB";

function ComponentA() {
  return (
    <div>
      <ComponentB />
    </div>
  );
}

export default ComponentA;
```
ComponentB:
```javascript
import React from "react";

import ComponentC from "./ComponentC";

function ComponentB() {
  return (
    <div>
      <ComponentC />
    </div>
  );
}

export default ComponentB;
```
ComponentC:
```javascript
import React from "react";
import { UserContext } from "./App";

function ComponentC() {
  return (
    <UserContext.Consumer>
      {user => <div>User context value {user}</div>}
    </UserContext.Consumer>
  );
}

export default ComponentC;
```

## 多个context
源码地址：https://codesandbox.io/s/react-contexs-k39sk?file=/src/index.js

App:
```javascript
import React from "react";
import "./styles.css";
import ComponentA from "./ComponentA";

export const UserContext = React.createContext("");
export const OtherContext = React.createContext("");

export default function App() {
  return (
    <div className="App">
      <UserContext.Provider value={"111"}>
        <OtherContext.Provider value={"sss"}>
          <ComponentA />
        </OtherContext.Provider>
      </UserContext.Provider>
    </div>
  );
}
```
ComponentC:
```javascript
import React from "react";
import { UserContext, OtherContext } from "./App";

function ComponentC() {
  return (
    <div>
      <UserContext.Consumer>
        {user => (
          <OtherContext.Consumer>
            {other => (
              <div>
                111User context value {user}, other value {other}
              </div>
            )}
          </OtherContext.Consumer>
        )}
      </UserContext.Consumer>
    </div>
  );
}

export default ComponentC;
```
## useContext

修改为useContext写法
源码：https://codesandbox.io/s/hook-usecontex-w23ux?file=/src/App.js
ComponentC:
```javascript
import React, { useContext } from "react";
import { UserContext, OtherContext } from "./App";

function ComponentC() {
  const user = useContext(UserContext);
  const other = useContext(OtherContext);
  return (
    <div>
      {user}-{other}
    </div>
  );
}

export default ComponentC;
```
