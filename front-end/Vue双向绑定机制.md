# Vue双向绑定

> Vue2.0

> 通过Object.defineProperty方法来监听对象中的属性变化，这种方法需要通过循环遍历对象中的属性来增加监听，并且对添加监听之后增加的属性无法监听，需要通过Vue.set()方法来添加监听

```JavaScript
let obj = {
     name:''
}
let newObj = {...obj}
Object.defineProperty(obj,'name',{
   get() {
         return newObj.name
   },
   set(v) {
         if(v === obj.name) return
         newObj.name = v
         observer()
   }
})
function observer() {
   spanName.innerHTML = obj.name
   inputName.value = obj.name
}
inputName.oninput = function () {
   obj.name = inputName.value
}
```

> Vue3.0
>
> 通过proxy方法来监听对象的变化，这种方法不需要循环遍历对象的属性，直接监听对象。

```
let obj = {}
  obj = new Proxy(obj, {
    get(target, p,) {
      return target[p]
    },
    set(target, p, value) {
      target[p] = value
      observer()
    }
  })

  function observer() {
    spanName.innerHTML = obj.name
    inputName.value = obj.name
  }

  inputName.oninput = function () {
    obj.name = inputName.value
  }
```

