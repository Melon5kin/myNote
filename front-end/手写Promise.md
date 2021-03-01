# 手写Promise

> 静态变量

```javascript
class MyPromise{
    static PENDING = 'pending'//准备状态
  	static FULFUILLED = 'fulfilled'//实现状态
  	static REJECTED = 'rejected'//拒绝状态
}
```

> 构造函数

```javascript
constructor(executor){
    this.status = MyPromise.PENDING //初始化Promise对象状态
    this.callback = [] //初始化存储异步执行函数的数组
    executor(this.resolve.bind(this),this.reject.bind(this)) // 将原型链上的resolve、reject函数绑定实例对象后注入执行函数并调用
}
```

> resolve方法

```javascript
resolve (value) {
    if (this.status === MyPromise.PENDING) { // 只有在状态为准备状态时调用函数才有效
      this.status = MyPromise.FULFUILLED
      this.value = value
      setTimeout(() => { //保证即使executor函数调用resolve是异步的，onResolved函数的调用也在下一轮任务队列中
        this.callback.forEach(callback => { //因为executor函数可能是异步的在then函数调用后执行
          callback.onResolved(value) // 因此当调用resolve方法时调用在then函数执行时存入的回调数组
        })
      })
    }
}
```

> reject 方法

```javascript
reject (reason) {
    if (this.status === MyPromise.PENDING) {
      this.status = MyPromise.REJECTED
      this.reason = reason
      setTimeout(() => {
        this.callback.forEach(callback => {
          callback.onRejected(reason)
        })
      })
    }
  }
```

> then方法

```javascript
then (onResolved, onRejected) {
    if (typeof onResolved !== 'function') {
      onResolved = () => this.value //当未为当前Promise对象的then方法传递方法时，将当前值保存传递给
    }                               //下一个Promise
    if (typeof onRejected !== 'function') {
      onRejected = () => this.reason
    }
    let promise = new MyPromise((resolve,reject) => {//返回Promise对象保证链式调用
      if (this.status === MyPromise.PENDING) {// 若当前状态为准备状态，则说明executor函数内执行改
        this.callback.push({  		//变状态函数时为异步的，then方法先执行，此时将需要调用的回调函数
          onRejected:reason => {//存入回调数组，当改变状态函数异步执行时会调用回调数组中的函数
            this.parse(promise,onRejected(reason),resolve,reject)
          },
          onResolved: value => {
            this.parse(promise,onResolved(value),resolve,reject)
          }
        })
      }
      if (this.status === MyPromise.FULFUILLED) {
        setTimeout(() => {
          this.parse(promise,onResolved(this.value),resolve,reject)
        })
      }
      if (this.status === MyPromise.REJECTED) {
        setTimeout(() => {
          this.parse(promise,onRejected(this.reason),resolve,reject)
        })
      }
    })
    return promise
  }
```

> parse方法

```javascript
parse (promise,result, resolve, reject) {
    try{
      if (promise === result) { //禁止在then中的方法返回自身这个Promise
        throw new Error('error msg')
      }
      if (result instanceof MyPromise) {//如果then中的处理函数返回值为Promise，则传递给下一个then
        result.then(resolve,reject)
      }else {
        resolve(result)
      }
    }catch (e) {
      reject(e)
    }
  }
```

> 静态方法

```javascript
static resolve (value) {
    return new MyPromise((resolve, reject) => {
      if (value instanceof MyPromise) {
        value.then(resolve,reject)
      }else {
        resolve(value)
      }
    })
  }
  static reject (reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason)
    })
  }

  static all (promises) {
    return new MyPromise((resolve,reject) => {
      const values = []
      promises.forEach(promise => {
        promise.then(value => {
          values.push(value)
          if (values.length === promises.length) { //只有当数组中的所有Promise状态都为实现时才会
            resolve(values)                        //将当前对象状态改变为fulfilled
          }
        },reason => {
          reject(reason)
        })
      })
    })
  }

  static race (promises) {
    return new MyPromise((resolve,reject) => {
      promises.forEach(promise => { //遍历执行所有Promise的executor，最先执行的改变当前返回值状态
        promise.then(value => {
          resolve(value)
        },reason => {
          reject(reason)
        })
      })
    })
  }
```

