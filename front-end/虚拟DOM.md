## 虚拟DOM

> 虚拟DOM是依靠h函数（渲染函数）来生成的一种描绘dom结构的对象，vue中进行dom的修改时，会先生成一个新的虚拟dom，与之前旧的虚拟dom进行比对，更新旧的虚拟dom，最后再将修改后的虚拟dom更新到真实dom。

### 一、虚拟DOM的生成

**h函数**

* h函数接受三个参数：sel（选择器，代表节点的标签）、data（保存了标签上的属性）、c（该参数可能接收文本、h函数、数组）

* 根据接受参数的不同h函数会vnode方法返回不同的虚拟dom

  * vnode方法接受五个参数：sel（选择权）、data（标签上的属性）、children（标签的子节点）、text（标签的文本节点）、elm（表示标签在真实dom中的节点）

  * h函数内部会根据接收参数的不同来给vnode方法传参

### 二、虚拟DOM的更新
> 虚拟DOM的更新依赖于diff算法的比对，diff算法主要由以下函数来实现
**patch函数**
* patch函数接受两个参数： oldNode（旧的虚拟dom或真实dom）、newNode（新的虚拟dom）
	
  * patch函数执行时首先判断oldNode是否为真实dom，若为真实dom则通过vnode方法将oldNode转化为虚拟dom
  * 其次，判断新旧虚拟dom是否为相同的节点（sel选择器相同并且key相同，在vue中还需要判断是否有相同的data，input的类型是否相同），若不为同一个对象则暴力拆除旧虚拟dom的当前节点替换为新虚拟dom的当前节点（过程为：利用insertBefore方法先将新节点插入到旧节点之前，再删除旧节点。）
  * 若为同一节点，则先判断新虚拟dom的子节点是否为文本节点，若为文本节点则可以直接将旧节点的elm.innerHTML替换为新的文本节点
  * 若新虚拟dom的子节点不为文本节点，则判断旧节点的子节点是否为文本节点，若为文本节点，则将旧节点的e l m.innerHTML清空然后遍历新虚拟dom的children将每个子节点通过createElement方法生成的真实元素放入到旧节点的elm中去。
  * 若新虚拟dom和旧虚拟dom都存在children时则通过updateChildren方法进行精细化比较。
** updateChildren函数**
* updateChildren函数接收三个参数： 旧虚拟dom的当前节点、旧节点的children、新节点的children
* 子节点的更新依靠四个指针的移动
	* oldStartIdx: 指向旧节点children的开始处，只能增加
	* oldEndIdx：指向旧节点children的结尾处，只能减少
	* newStartIdx：指向新节点children的开始处，只能增加
	* newEndIdx：指向新节点children的结尾处，只能减少
* 子节点的比对有四种命中方式（指针所指向节点相同则为命中）
	* 新前和旧前（newStartIdx和oldStartIdx 以此类推）命中时将两个节点进行patch并将两个对应指针移动
	* 新后和旧后 命中时将两个节点进行patch并将两个对应指针移动
	* 新后和旧前	命中时将两个节点进行patch，然后将旧节点放到旧后的后面，并将两个对应指针移动
	* 新前和旧后 命中时将两个节点进行patch，然后将旧节点放到旧前的前面，并将两个对应指针移动
* 若四种方式都未命中，则将旧节点的key按照索引建立一个hash表（keyMap），然后循环新节点的children，查看keyMap中是否含有和当前子节点相同的节点，若有则进行patch，然后放入到旧前到前面，并将旧节点当中的这个节点标记为undefinded（表示该节点已经被处理过，之后不需要再进行处理）
* 以上操作处理完后，只剩下新增和删除节点，于是进行判断。若oldChildren或newChildren中任何一个到startIdx <= endIdx,说明需要进行新增或删除操作，然后进行对应操作。

  
  

