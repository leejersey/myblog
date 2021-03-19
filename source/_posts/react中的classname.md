---
title: react中的classname
date: 2021-03-18 09:44:36
tags: react
---

工作中经常会碰到需要给jsx中加多个classname.

## es6模板字符串方法
```
className={`title ${index === this.state.active ? 'active' : ''}`}
```

## 字符串连接
```
className={["title", index === this.state.active?"active":null].join(' ')}
```

## classnames
```
var classNames = require('classnames');

var Button = React.createClass({
  // ...
  render () {
    var btnClass = classNames({
      btn: true,
      'btn-pressed': this.state.isPressed,
      'btn-over': !this.state.isPressed && this.state.isHovered
    });
    return <button className={btnClass}>{this.props.label}</button>;
  }
});
```