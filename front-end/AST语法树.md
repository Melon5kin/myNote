# AST语法树

> Vue底层原理中对模版进行编译的过程中首先是将template模版转化为AST语法树，然后再将AST语法树转化为render函数生成虚拟dom（用作之后组件更新后先进行diff算法比对修改虚拟dom最后再更新真实dom）
>
> 实现简易的parse方法：

```javascript
function parse(templateString) {
    const startTagReg = /^\<([a-z]+[1-6]?)(\s[^\<]+)?\>/ //正则匹配开始标签
    const endTagReg = /^\<\/([a-z]+[1-6]?)\>/	//正则匹配结束标签
    const contentReg = /^([^\<]+)\<\/([a-z]+[1-6]?)\>/ //正则匹配内容
    let index = 0 //指针指向当前已处理进度的索引
    let tagStack = [] //标签栈
    let astStack = [{ children: [] }] //ast语法栈
    while (index < templateString.length - 1) { //遍历模版字符串
        let ts = templateString.substring(index) //获取尚未处理的模版字符串
        if (startTagReg.test(ts)) { // 获取到开始标签
            let tag = ts.match(startTagReg)[1] // 获取匹配到的标签内容
            let attrs = ts.match(startTagReg)[2] // 获取匹配到的属性内容
            let attrsLength = attrs ? attrs.length : 0
            index += tag.length + attrsLength + 2 // 进度索引向后移动整个开始标签的长度
            let parseAttrs = parseAttrsString(attrs) // 处理属性字符串，并获得属性数组
            // 遇到开始标签时将标签推入标签栈，将标签对应的ast对象推入ast栈
            tagStack.push(tag)
            astStack.push({ tag: tag, type: 1, children: [], attrs: parseAttrs })
        } else if (endTagReg.test(ts)) {// 匹配到结束标签
            let endTag = ts.match(endTagReg)[1] // 获取结束标签内容
            index += endTag.length + 3 // 进度索引向后移动结束标签长度
            if (endTag === tagStack[tagStack.length - 1]) {// 与当前标签栈栈顶标签进行比对检查标签是否闭合，若未闭合则报错
                // 遇到结束标签时将标签栈顶顶元素出栈，将当前标签对应的ast对象push到父元素对应ast数组的children属性中
                tagStack.pop()
                let ast_pop = astStack.pop() // 获取当前已完整捕获到的标签ast对象
                astStack[astStack.length - 1].children.push(ast_pop) // push到当前ast栈栈顶元素（即捕获元素的父元素）
            } else {                                                 // 的children属性上
                throw new Error('Invalid Tag')
            }
        } else if (contentReg.test(ts)) {// 匹配到标签内容
            let content = ts.match(contentReg)[1] // 获取内容
            if (!/^\s+$/.test(content)) { //判断是否为空字符串
                // 匹配到标签内容时将内容push到ast栈中对应标签的children中
                astStack[astStack.length - 1].children.push({ 'text': content, type: 3 })
            }
            index += content.length //进度指针向后移动内容长度
        }
        else {
            index++
        }
    }
    return astStack[0].children[0]
}
```

> 属性处理方法：

```javascript
function (attrsString) {
    let result = []
    let index = 0 // 进度索引
    let flag = false // 当前遍历到的空格是否在属性值内部的判断标识
    if (!attrsString) return []
    for (let i = 0; i < attrsString.length; i++) { // 遍历属性字符串
        if (attrsString[i] === '"') { // 当前字符若为“则说明当前字符所处环境发生改变判断标识需要发生改变
            flag = !flag
        } else if (attrsString[i] === ' ' && !flag) { // 若当前字符为空格且判断标识为false（当前所处环境不在属性值内部）
            if (/^\s?$/.test(attrsString.substring(index, i))) { // 可能为空字符串
                continue
            }
            let attr = attrsString.substring(index, i).trim() // 截取遍历到的完整属性键值对
            index = i // 改变进度索引的位置
            result.push(attr) // 将获取到的属性放入数组
        }
    }
    result.push(attrsString.substring(index)) //最后一个属性可能因为最后没有空格而没有被捕获到，手动将它push进去
    result = result.map(attr => { // 处理数组中的每一个属性字符串
        let temp = attr.match(/^(.+)="(.+)"$/) // 匹配属性字符串中的键和值
        return { // 将键和值映射出去
            name: temp[1],
            value: temp[2]
        }
    })
    return result
}
```

