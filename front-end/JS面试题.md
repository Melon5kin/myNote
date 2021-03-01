## 一、 

> 如何设置变量的值使得满足以下条件

```javascript
a = ?
if (a == 1 && a == 2 && a== 3) {
  console.log('1')
}
```

* 通过数据劫持

```javascript
window.a = ''
let i = 0
Object.defineProperty(window,'a',{
  get() {
    return ++i
  }
})
```

* 通过重写toString方法

> 由于当变量a为一个对象，并与数字进行比较时，对象类型会通过toString方法先转化为字符串，再通过Number（）方法转化为数字。当对象中存在toString方法时会优先使用对象的toString方法。

```javascript
let a  = {
  i:0,
  toString() {
    return ++this.i
  }
}
if (a == 1 && a == 2 && a== 3) {
  console.log('1')
}
```

