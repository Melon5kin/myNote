# Vue 组件通信

## 一、自定义dispatch和broadcast

> dispatch 将事件向上分发给每个parent

```javascript
Vue.prototype.$dispatch = function (eventName, value) {
    const parent = this.$parent
    while(parent){
        parent.$emit(eventName, value)
        parent = parent.$parent
    }
}
```

> broadcast 将事件向下触发每个children

```JavaScript
Vue.prototype.$broadcast = function (eventName, value) {
    const broadcast = (children) => {
        children.forEach(child => {
            child.$emit(eventName, value)
            if (child.$children) {
                broadcast(child.$children)
            }
        })
    }
    broadcast(this.$children)
}
```

## 二、eventBus

> eventBus会触发所有同名事件

```
Vue.prototype.$eventBus = new Vue() // 利用vue示例中自带的$on 、 $emit 方法来实现全局触发事件
```

