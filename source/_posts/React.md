---
title: React 学习
date: 2025-01-11
tags: [React, 程序员, 技术]
categories: [技术分享]
---
# React 学习  
## jsx
- JSX语法是一种JS的扩展语法，它可以在JS中直接使用HTML标签，而不需要像原生一样使用html字符串
- JSX语法必须使用一个根标签包裹所有的内容
- JSX语法中可以混入JS表达式，但是必须使用{}包裹起来
- JSX语法中可以使用注释，但是必须使用{}包裹起来
- JSX中class属性需要使用className代替，style属性需要使用对象形式：`style={{key: value}}`
- JSX中标签必须闭合
- 组件标签必须是大写字母开头

## React 基本渲染
1. 定义虚拟DOM
2. 渲染虚拟DOM到页面
```jsx
import React from 'react'
import ReactDOM from 'react-dom'
const VDOM = (
  <div>
    <h1>hello world</h1>
  </div>
) 

ReactDOM.render(<VDOM />, document.getElementById('root'))
// root 为html文件中的一个div标签 用来指定这些虚拟DOM渲染的目标对象
// 如果要重复渲染 需要给标签上加上key属性
```
-------------

## 组件渲染
1. 类相关的复习
2. 函数组件
2. 类组件
```jsx
class Person{
  // constructor 构造方法  每次new实例化对象时 会自动调用,如果没有js会调用一个默认的构造方法（隐式调用）
  // 构造方法内的this指向实例对象
  constructor(name,age){
    this.name = name
    this.age = age
  }
  // 实例方法 放在类的原型对象上
  say(){
    console.log(this.name)
  }
  //继承
  class Student extends Person{
    constructor(name,age,score){
      super(name,age) //super用来继承父类的属性  如果要写构造函数来继承父类属性则super必须放在最前面
      this.score = score 
    }
    //  子类可以对父类的方法重写 此时子类方法放在student的原型上 如果没有重写去调用，则调用的是父类原型上的方法
    say(){
      console.log(this.name,this.age,this.score)
    }
  }
}
```
 -------------
```jsx
 // 函数组件
const App = () => {
  return (
    <div>
      <h1>hello world</h1>
    </div>
  )
}   
//如果这个函数是普通函数 那么它就是函数组件 则函数内this指向undefined（严格模式下）

 // 类组件
class App extends React.Component {
  render() {
    return (
      <div>
        <h1>hello world</h1>
      </div>
    )
  }
}   
//类式组件 则函数内this指向实例对象  
ReactDOM.render(<App />, document.getElementById('root'))

// React 解析过程 
// 如果是函数组件 则直接调用函数
// 如果是类组件 则先实例化对象 再调用render方法
// 将虚拟DOM转换为真实DOM
```

-------------

## 三大基础

### state
```jsx
// 类组件
class App extends React.Component {
  // 构造方法
  constructor() {
    super()
    // 初始化state 只能放对象在state中
    this.state = {
      count: 0
    }
  }
  render() {
    // 渲染state中的数据
    const {count} = this.state
      // 改变state中的数据
      this.setState({
        count: 66
      }) //要通过setState方法来改变state中的数据

    return (
      <div>
        <h1>hello world</h1>
      </div>
    )
  }
}
```
### props
### ref

-----------

## 事件绑定

### js原生事件复习

```js
// 事件绑定
1. 行内事件绑定(React 与 vue用的都是这种方式)
<div>
  <button onclick="fun()">点我</button>
</div>

2. 对DOM对象绑定事件
const div = document.getElementById('div1')
div.onclick = function(){
  console.log('hello world')
}

3. 事件委托
// 事件委托
const ul = document.getElementById('ul1')
ul.onclick = function(e){
  console.log(e.target)
}
```
### React事件绑定
1. 类组件事件绑定
```jsx
class App extends React.Component {
  // 构造方法
  constructor() {
    super()
    // 初始化state 只能放对象在state中
    this.state = {
      count: 0
    }
  }
  // 事件处理函数
  handleClick() {
    console.log('hello world')
  }
  render() {
    // 渲染state中的数据
    const {count} = this.state
      // 改变state中的数据
      this.setState({
        count: 66
      }) //要通过setState方法来改变state中的数据

    return (
      <div>
        <h1>hello world</h1>
        {/* react 中的事件都是小驼峰形式而非原生中的onclick形式 且函数名后不能跟（） */}
        <button onClick={this.handleClick}>点我</button>
      </div>
    )
  }
}
```
