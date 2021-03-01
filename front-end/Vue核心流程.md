# Vue 核心流程

1.首先接收创建实例时传入的options

```javascript
class Vue{
    constructor(options){
        this.$el = options.el
        this.$data = options.data
        this.$options = options
    }
}
```

2.将$data中的数据通过代理能够通过this来访问

```javascript
proxyData(data){
    Object.keys(data).forEach(key => {
        Object.defineProperty(this,key,{
            get(){
                return this.$data[key]
            },
            set(newValue){
                if(newValue === this.$data[key]) return
                this.$data[key] = newValue
            }
        })
    })
}
```

3.创建Observer对象对每个$data中的对象进行双向绑定

```javascript
class Observer {
  constructor (data) {
    let realData = data
    if (typeof realData === 'function') {
      realData = realData() // 如果options中的data为函数则取函数的返回值
    }
    this.observe(realData)// 对数据绑定劫持
  }

  observe (data) {
    if (data && typeof data === 'object') {
      Object.keys(data).forEach(key => {
        this.defineReactive(data, key, data[key]) // 遍历数据进行响应式绑定
      })
    }
  }

  defineReactive (data, key, value) {
    this.observe(value) // 如果属性值为对象进行递归调用绑定所有属性
    const dep = new Dep() // 创建依赖对象，保存当前对象的依赖
    Object.defineProperty(data, key, {
      get () {
        const target = Dep.target        //每当编译模板时发现需要依赖数据时会数据的getter，此时将
        target && dep.addWatcher(target) //此watcher添加到当前数据的依赖中（利用闭包）
        return value                      
      },
      set:(v) => {
        if (v === value) return
        this.observe(v) // 设置的新值可能为对象，需要再次进行响应式绑定
        value = v
        dep.notify() // 通知当前数据的所有依赖进行更新
      }
    })
  }
}
```

4.将页面中的模板字符串、指令等进行编译

```javascript
class Compile {
  constructor (el, vm) {
   	this.el = this.isElementNode(el) ? el : document.querySelector(el)//判断传入el是否为节点
    this.vm = vm
    const fragment = this.compileFragment(this.el) // 创建插入el中内容的文档片段
    this.compile(fragment) // 编译文档片段
    this.el.appendChild(fragment) // 将编译完成后的文档片段插入到el中
  }

  compile (fragment) {
    const childNodes = Array.from(fragment.childNodes) // 获取文档片段中的子节点列表
    childNodes.forEach(childNode => { //  遍历分别处理元素节点和文本节点
      if (this.isElementNode(childNode)) {
        this.compileElement(childNode)
      } else if (this.isTextNode(childNode)) {
        this.compileText(childNode)
      }
      if (childNode.childNodes && childNode.childNodes.length) {
        this.compile(childNode) //如果节点存在子节点则递归编译子节点
      }
    })
  }

  compileElement (node) {
    const attributes = Array.from(node.attributes) //获取元素节点的属性
    attributes.forEach(attr => {
      const { name, value } = attr
      if (this.isDirective(name)) { //判断属性是否为指令
        const [, directive] = name.split('-') // 获取对应指令
        const [compileKey, eventName] = directive.split(':') //如果是v-on或v-bind绑定事件则获取事件名
        utils[compileKey](node, value, this.vm, eventName)//调用utils中的方法编译对应指令的内容
      }else if (this.isEventDirective(name)) {
        const [,directive] = name.split('@') // 处理@等缩写的指令
        utils['on'](node,value,this.vm,directive)
      }
    })

  }

  isEventDirective (name) {
    return name.startsWith('@')
  }
  isDirective (name) {
    return name.startsWith('v-')
  }

  compileText (node) {
    let content = node.textContent
    if (/\{\{(.+)\}\}/.test(content)) { //匹配模板字符串格式
      utils['text'](node,content,this.vm) //调用utils中编译文本节点的方法
    }
  }

  compileFragment (el) {
    const f = document.createDocumentFragment() // 创建文档片段
    let firstChild
    while (firstChild = el.firstChild) { //将el指定的标签下的所有元素插入到文档片段中
      f.appendChild(firstChild)
    }
    return f
  }

  isElementNode (el) {
    return el.nodeType === 1
  }

  isTextNode (el) {
    return el.nodeType === 3
  }
}
```

5.utils

```javascript
const utils = {
  getValue (expr, vm) { //获取表达式对应的值
    return vm.$data[expr.trim()]
  },
  setValue (expr, vm, newValue) { //设置表达式对应的值
    vm.$data[expr.trim()] = newValue
  },
  model (node, value, vm) { //编译v-model指令的模板
    const initValue = this.getValue(value, vm) //获取初始值
    new Watcher(value,vm,(newValue) => { //创建当前value的watcher实例
      this.modelUpdater(node,newValue)
    })
    node.addEventListener('input', (e) => { //添加当前节点的input监听事件
      const newValue = e.target.value
      this.setValue(value, vm, newValue)
    })
    this.modelUpdater(node,initValue) //更新当前节点的视图
  },
  text (node, value, vm) { //编译模板字符串或v-text指令的模板
    let result
    if (value.includes('{{')) { //编译模板字符串
      result = value.replace(/\{\{(.+)\}\}/g,(...args) => {
        const expr = args[1]
        new Watcher(expr,vm,(newValue) => { //创建当前value的watcher实例
          this.textUpdater(node,newValue)
        })
        return this.getValue(args[1],vm)
      })
    }else { //编译v-text
      new Watcher(value,vm,(newValue) => {
        this.textUpdater(node,newValue)
      })
      result = this.getValue(value,vm)
    }
    this.textUpdater(node,result)//更新视图
  },
  on (node, value, vm, eventName) {
    const fn = vm.$options.methods[value]
    node.addEventListener(eventName,fn.bind(vm),false)
  },
  textUpdater (node, value) {
      node.textContent = value
  },
  modelUpdater (node, value) {
    node.value = value
  }
}
```

6.Watcher和Dep

```javascript
class Watcher {
  constructor (expr,vm,cb) {
    this.expr = expr
    this.vm = vm
    this.cb = cb
    this.oldValue = this.getOldValue()
  }

  getOldValue () {
    Dep.target = this //通过全局变量记录当前watcher实例
    const oldValue = utils.getValue(this.expr,this.vm) //调用getter方法将当前watcher添加到对应数据的Dep依赖上
    Dep.target = null
    return oldValue
  }

  update () {//更新视图
    const newValue = utils.getValue(this.expr,this.vm)
    if (newValue !== this.oldValue) {
      this.cb(newValue)
    }
  }
}
class Dep {
  constructor () {
    this.collect = []
  }

  addWatcher (watcher) {
    this.collect.push(watcher)
  }

  notify () { //通知所有watcher进行视图更新
    this.collect.forEach(w => w.update())
  }
}
```

