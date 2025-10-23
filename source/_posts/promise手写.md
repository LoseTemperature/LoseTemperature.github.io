---
title: Promise手写
date: 2025-01-11
tags: [浏览器, 程序员, 技术]
categories: [技术分享]
---

# Promise 手写实现教程

## 1. 基础概念与示例

### Promise 基本用法

```javascript
// promise 示例
const p = new Promise((resolve, reject) => {
  resolve(1)
})
// promise手写思路
// 首先，Promise是为了解决多个异步操作多层嵌套时，最终的结果依靠回调函数来判断 ，但多层嵌套会让其难以排查和维护（也就是回调地狱），所以Promise 实质上是一个表示 一个或者多个异步操作函数 的状态结果（成功或者失败）的对象 【1.你在用ai写一个网页（异步任务）, 你不会立刻得到网页的效果（可能成功，可能失败），但ai会给你一个本次任务执行中的状态（loading|成功|失败）这个状态就是promise对象，那初始状态就是loading(也就是promise中的pending), 然后ai会执行任务（也就是异步任务），最终ai会返回一个结果并且这个结果不可能改变（因为本质上一个任务要么成功要么失败）那会2.返回两个promise的状态 resolve|reject 来告知你这个异步操作到底是成功了还是失败了，如果成功失败了怎么做】

// then示例
p.then((res) => { }, (err) => { })
// then的手写思路
// 首先，then 需要接收两个回调 onFulfilled|onRejected 也就是成功|失败的回调，然后，then会返回一个新的promise
// 然后就是主要面临的问题:
// 1. 什么时候调用这两个回调？ 显然是上个promise为fulfilled|rejected 状态时调用对应的回调，那么如果上一个promise没有状态呢？（异步操作可以会在then时还没有执行）那么就需要把这个功能从then抽离出来假设为run函数,在run中调用回调函数（onFulfilled|onRejected），then记录当前的状态的内部变量hendlers(同时有多个挂起的异步任务)，在then中调用run，run执行一下如果是状态是pending,则什么也不做，等待promise状态改变再调用run,也就解决了异步状态挂起的问题 2.then是可以链式调用的，也就是说run可能需要去执行多个回调，所以headler最好是个数组对象
// 2. 什么时候去设置then返回的promise的状态并执行相应的回调？（1）当传入的数据不是一个函数时，此时then返回的promise状态应该与上级promise状态保持一致（穿透）（2）传入的是函数时，需要用trycatch去检查是否出错，出错状态改为reject,不出错拿到函数的结果，然后状态改为resolve（3）传入的是另一个promise，此时需要判断传入的是不是promise（这个promise不能只限于手写的和官方的promise,只要符合promise标准就要识别，要有泛用性） 
// 3. then里的函数应该要放入微队列中，怎样处理?

// 代码

// 定义全局常量 
const PENDING = "pending" //定义常量方便修改
const RESOLVE = "fulfilled"
const REJECT = "rejected"

class MyPromise {
  //对应思路1. 先传入一个函数，然后执行

  // 然后promise需要有两个变量1. 状态 2. 返回结果
  #state = "pending" // # 变量 表示 私有变量不能被外部修改（返回一个结果并且这个结果不可能改变（因为本质上一个任务要么成功要么失败））
  #result = undefined
  #run = undefined
  #handlers = []
  constructor(executor) {

    // // 对应思路2.这两个函数是用来返回当前任务的状态做成函数来解决成功|失败后 返回的结果 
    const resolve = (data) => {//所以这两个函数要做两个内容1.设置当前的状态 2.返回对应的结果 
      // if (this.#state !== "pending") return //保证状态只修改一次，一但修改了状态就不能再变动了（只针对本次任务）
      // this.#state = "fulfilled"
      // this.#result = data
      this.#changeState(RESOLVE, data)
    }
    const reject = (reason) => {
      // if (this.#state !== "pending") return
      // this.#state = "rejected"
      // this.#result = reason
      this.#changeState(REJECT, reason)
    }
    try {
      executor(resolve, reject)
    } catch (err) {  //如果函数执行时出错，则状态也设置为rejected ,但没法捕获异步错误
      reject(err)
    }

    #changeState(state, result){  //把resolve 和reject 封装为一个函数复用
      if (this.#state !== PENDING) return
      this.#state = state
      this.#result = result

      this.#run()//配合then处理异步调用状态挂起（pending）问题，在异步操作有结果时重新调用回调
    }
    #isPromiseLike(value){
      if (value !== null && (typeof value === 'object' || typeof value === 'function')) {
        return typeof value.then === 'function';
      }
      return false;
    }
    #runMicroTask(func){
      queueMicrotask(func);
    }
    #runOne(callback, resolve, reject){
      this.#runMicroTask(() => {
        if (typeof callback !== 'function') {
          const settled = this.#state === RESOLVE ? resolve : reject
          settled(this.#result)
          return
        }
        try {
          const data = callback(this.#result)
          if (this.#isPromiseLike(data)) {
            data.then(resolve, reject)// 根据新的promise状态决定当前promise状态
          } else {
            resolve(data)
          }

        } catch (error) {
          reject(error)
        }
      })
    }

    #run(){
      if (this.#state === PENDING) return
      while (this.#handlers.length) {
        const { onFulfilled, onRejected, resolve, reject } = this.#handlers.shift()
        if (this.#state === RESOLVE) {
          // if (typeof onFulfilled === 'function') {
          //   try {
          //     const data = onFulfilled(this.#result)
          //     resolve(data)
          //   } catch (error) {
          //     reject(error)
          //   }

          // } else {
          //   resolve(this.#result)
          // }

          // 简化利用上面的代码
          this.#runOne(onFulfilled, resolve, reject)

        } else {
          // if (typeof onRejected === 'function') {
          //   try {
          //     onRejected(this.#result)
          //     const data = onRejected(this.#result)
          //     resolve(data)
          //   } catch {
          //     reject(error)
          //   }

          // } else {
          //   reject(this.#result)
          // }
          this.#runOne(onRejected, resolve, reject)
        }
      }
    }

    then(onFulfilled, onRejected){
      return new MyPromise((resolve, reject) => {
        this.#handlers.push({
          onFulfilled,
          onRejected,
          resolve,
          reject
        })

        // if (this.#state === RESOLVE) {
        //   onFulfilled(this.#result)
        // } else if (this.#state === REJECT) {
        //   onRejected(this.#result)
        // } else {
        //   //处理异步操作
        // }   || 优化为下面的run方法
        // v
        this.#run()

      })
    }
  }
}
```

## 总结

这个 Promise 实现展示了从基础概念到完整实现的全过程，特别体现了代码重构的重要性：

1. **状态管理**：通过私有变量确保状态的不可逆性
2. **异步处理**：通过 handlers 数组和 run 方法处理异步回调
3. **代码优化**：从冗余的条件分支到精简的 runOne 方法
4. **标准兼容**：通过 isPromiseLike 方法支持 Promise/A+ 规范

通过这种循序渐进的方式，我们不仅实现了一个功能完整的 Promise，更重要的是理解了其背后的设计思想和优化过程。