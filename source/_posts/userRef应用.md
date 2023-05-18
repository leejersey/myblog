---
title: userRef应用
date: 2023-05-18 10:24:42
tags:
---

### 1.useRef属性
*   导入 useRef 函数从 react 中
*   创建 ref 对象 const ref = useRef(null)
*   给需要获取的标签上 ref={ref} 绑定 ref 对象
*   渲染完毕后，可以通过 ref.current 获取 dom 元素

```
// 测试 自定义hook
import { usePoint } from "./utils/hooks"
import { useRef } from "react" //1.
const App = () => {
  const point = usePoint()
  const inputRef = useRef(null) // null表示初始值  inputRef.current 初始值为null // 2.
  const getInputValue = () => {
    // 4.
    console.log(inputRef.current.value)
  }
  return (
    <div>
      <p>
        当前的鼠标的坐标是{point.pageX}, {point.pageY}
      </p>
      {/* 3. */}
      <input ref={inputRef} type="text" />
      <button onClick={() => getInputValue()}>获取dom对象</button>
    </div>
  )
}
export default App


```

### 2.useRef 的深度用法

本质上，useRef 就像是可以在其 .current 属性中保存一个可变值的“盒子”。

你应该熟悉 ref 这一种访问 DOM 的主要方式。如果你将 ref 对象以 <div ref={myRef} /> 形式传入组件，则无论该节点如何改变，React 都会将 ref 对象的 .current 属性设置为相应的 DOM 节点。

然而，useRef() 比 ref 属性更有用。它可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。

这是因为它创建的是一个普通 Javascript 对象。而 useRef() 和自建一个 {current: ...} 对象的唯一区别是，useRef 会在每次渲染时返回同一个 ref 对象。


*   不论组件如何更新，ref 的存储值都不会发生变化，除非你设置它
*   useState 中的状态 在函数中永远都是最新的状态

如：如何来获取上一次的状态

```
// 获取上一次的状态

export const usePrevStatus = (state) => {
  const valueRef = useRef(state) // 生成一个ref对象
  useEffect(() => {
    valueRef.current = state // 赋值最新的值
  }, [state])

  return valueRef.current // 返回这个值
}

```

使用：

```
  const [count, setCount] = useState(0)
  const prevValue = usePrevStatus(count)
  console.log(prevValue)

```

### 3. 发送验证码:
  
代码:

```
import "./App.css"
import { Button } from "antd"
import { useState, useRef, useEffect } from "react"
const App = () => {
  const timerCount = 60 // 默认60秒
  const [count, setCount] = useState(timerCount)
  const timerRef = useRef(null) // 记录时间的定时器
  const cutCount = () => {
    setCount((prevState) => prevState - 1) // 为什么这里要用函数- 如果用count 发现count闭包了 不会发生变化了
  }
  const sendCode = () => {
    // 要发送验证码
    cutCount()
    timerRef.current = setInterval(cutCount, 1000)
  }
  useEffect(() => {
    if (count === 0) {
      clearInterval(timerRef.current) // 清空定时器
      setCount(timerCount) // 重新将计时器设置为60秒
    }
  }, [count])

  return (
    <div class>
      <Button
        type="primary"
        disabled={count < timerCount}
        onClick={count === timerCount ? sendCode : null}
      >
        {count === timerCount ? "发送验证码" : `还剩${count}秒`}
      </Button>
    </div>
  )
}

export default App

```