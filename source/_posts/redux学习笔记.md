---
title: redux学习笔记
date: 2018-10-22 21:18:40
tags: redux
---

## redux基础

```jsx
import { createStore } from 'redux'

// 这就是reducer处理函数，参数是状态和新的action
function counter(state=0, action) {
  // let state = state||0
  switch (action.type) {
    case '加机关枪':
      return state + 1
    case '减机关枪':
      return state - 1
    default:
      return 10
  }
}
// 新建保险箱
const store = createStore(counter)
// console.log
const init = store.getState()
console.log(`一开始有机枪${init}把`)
function listener(){
  const current = store.getState()
  console.log(`现在有机枪${current}把`)
}
// 订阅，每次state修改，都会执行listener
store.subscribe(listener)
// 提交状态变更的申请
store.dispatch({ type: '加机关枪' })
store.dispatch({ type: '加机关枪' })
store.dispatch({ type: '加机关枪' })
store.dispatch({ type: '减机关枪' })
store.dispatch({ type: '减机关枪' })
```

## 解决异步redux-trunk


```jsx
const ADD_GUN = '加机关枪'
const REMOVE_GUN = '减机关枪'
// 这就是reducer处理函数，参数是状态和新的action
export function counter(state=0, action) {
  // let state = state||0
  switch (action.type) {
    case ADD_GUN:
      return state + 1
    case REMOVE_GUN:
      return state - 1
    default:
      return 10
  }
}
export function addGun(){
  return { type: ADD_GUN }
}
export function removeGun(){
  return { type: REMOVE_GUN }
}
// 延迟添加，拖两天再给
export function addGunAsync(){
  // thunk插件的作用，这里可以返回函数，
  return dispatch => {
    setTimeout(() => {
      // 异步结束后，手动执行dispatch
      dispatch(addGun());
    }, 2000);
  };

}
```


## react-redux
index.js
```jsx
import ReactDOM from 'react-dom'
import { createStore, applyMiddleware, compose} from 'redux'
import thunk from 'redux-thunk'
import { counter } from './index.redux'
import { Provider } from 'react-redux';
import App from './App'

const store = createStore(counter, compose(
  applyMiddleware(thunk),
  window.devToolsExtension ? window.devToolsExtension() : f => f
))
ReactDOM.render(
  (
    <Provider store={store}>
      <App />
    </Provider>
  ),
  document.getElementById('root'))
```
App.js

```jsx
import { connect } from 'react-redux'
import { addGun, removeGun, addGunAsync } from './index.redux'

class App extends React.Component{
  render(){
    // num addGun，removeGun，addGunAsync都是connect给的，不需要手动dispatch
    return (
      <div>
        <h2>现在有机枪{this.props.num}把</h2>
        <button onClick={this.props.addGun}>申请武器</button>
        <button onClick={this.props.removeGun}>上交武器</button>
        <button onClick={this.props.addGunAsync}>拖两天再给</button>
      </div>
    ) 
  }
}

const mapStatetoProps = (state) => {
    return {
        num: state
    }
}

const actionCreators = { addGun, removeGun, addGunAsync }

App = connect(mapStatetoProps, actionCreators)(App)

export default App;
```

## 装饰器模式

webpack配置

```markdown
npm install babel-plugin-transform-decorators-legacy --save-dev
```

.babelrc配置

```json
{
  "plugins": ["transform-decorators-legacy"]
}
```

App.js

```jsx
import { connect } from 'react-redux'
import { addGun, removeGun, addGunAsync } from './index.redux'

// 装饰器模式
@connect(
  state=>({ num: state}),
  {addGun, removeGun, addGunAsync}
)
class App extends React.Component{
  render(){
    // num addGun，removeGun，addGunAsync都是connect给的，不需要手动dispatch
    return (
      <div>
        <h2>现在有机枪{this.props.num}把</h2>
        <button onClick={this.props.addGun}>申请武器</button>
        <button onClick={this.props.removeGun}>上交武器</button>
        <button onClick={this.props.addGunAsync}>拖两天再给</button>
      </div>
    ) 
  }
}
export default App;
```