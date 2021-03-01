# Vue生命周期

### Vue实例创建阶段

> beforeCreated

此时Vue实例初始化完一些事件和生命周期，还未将data、methods等options属性中的数据绑定到实例上，因此此时无法获取数据。

> created

此时Vue实例已经将data、methods等options属性中的数据绑定到实例上，可以访问到这些数据。

> beforeMount

此时已经在内存中将页面中的模板字符串编译完成，但此时未将编译后的模板挂载到Vue实例上。因此，此时页面中的展示的数据仍然是未编译的模板字符串。

> mounted

此时已将编译后的模板挂载在Vue实例上，因此，此时页面上展示的数据为编译后的数据。

### Vue实例运行阶段

> beforeUpdate

当data中的数据发生改变时会触发该生命周期函数，但此时页面内容并没有更新。在这之后虚拟DOM重新渲染，并更新。

> updated

此时页面中的内容已经更新完成。

### Vue实例销毁阶段

> beforeDestroy

此时Vue实例还未开始销毁，实例中的data、methods、指令等数据还存在，并且可以使用。在这之后实例开始销毁。

> destroyed

此时Vue实例已经销毁，实例中的数据无法使用。