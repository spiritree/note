# element的应用
## 通过生命周期优化性能

使用element的Table组件从后端获取列表数据

![生命周期](/assets/FrontEnd/Component/element/vue-lifecycle.png)

使用方法：

1. created: axios的异步请求
2. beforeUpdate: 2个页面使用同一个子组件异步请求获取Table数据更新页面
3. nextTick: 更新数据后立即操作DOM（在mounted中）

## Table 组件的增删改

Vue是一个MVVM框架，我们传统写代码是命令式编程（代表库JQuery），拿到Table这个DOM之后就是命令式对DOM增删改。而我们现在用MVVM框架，只用关注data的变化和View层就好了，所以我们这里的增删改都是基于list这个数组来的。

!> 由于 JavaScript 的限制， Vue 不能检测以下变动的数组：<br>
当你利用索引直接设置一个项时，例如： vm.items[indexOfItem] = newValue
所以我们想改变table中第一条数据的值，通过this.list[0]=newValue这样是不会生效的。

解决方法：
```javascript
// 添加数据
this.list.unshift(this.temp); // 在表单头部添加数据
this.list.push(this.temp); // 在表单尾部添加数据

// 删除数据 
const index = this.list.indexOf(row); // 找到要删除数据在list中的位置
this.list.splice(index, 1); // 通过splice 删除数据

// 修改数据
const index = this.list.indexOf(row); // 找到修改的数据在list中的位置
this.list.splice(index, 1, this.updatedData); // 通过splice 替换数据 触发视图更新
```

## 如何在 Table 组件的每一行添加操作该行数据的按钮？

使用 Vue 的 [Scoped slot](https://cn.vuejs.org/v2/guide/components.html#Scoped-Slots) 即可：

![scope.png](/assets/FrontEnd/Component/element/element-scope.png)

```html
<el-table-column label="操作">
  <template scope="props">
    <el-button @click.native="showDetail(props.row)">查看详情</el-button>
  </template>
</el-table-column>
```

## 缓存Tab内容

假设我们有n个Tab选项，每个Tab都会向后端请求数据，但我们希望一开始只会请求当前的Tab数据，而且Tab来回切换的时候不会重复请求，只会渲染一次。首先我们想到的就是用`v-if` 这样的确能做到一开始不会加载后面的tab，但有一个问题，每次点击这个tab组件都会重新挂载一次，这时候我们就可以用到`<keep-alive>`了。

```html
<el-tabs v-model="activeTab">
  <el-tab-pane label="贷款项目详情" name="loanProjDetailComponent">
    <loanProjDetailComponent />
  </el-tab-pane>
  <el-tab-pane label="车贷费用" name="carFeeInfoComponent">
    <keep-alive>
      <carFeeInfoComponent v-if="activeTab=='carFeeInfoComponent'" />
    </keep-alive>
  </el-tab-pane>
  <el-tab-pane label="车辆信息" name="carInfoComponent">
    <keep-alive>
      <carInfoComponent v-if="activeTab=='carInfoComponent'" />
    </keep-alive>
  </el-tab-pane>
</el-tabs>
```

## 父组件向子组件传递数据
```html
<div id="demo">
  <!-- 这是父组件 -->
  <component is="myComponent" data-send-to-father="dataFromChild"></component>
</div>
```
```javascript
Vue.component('myComponent', {
	// 子组件要用props来声明想要获得的数据
  props: ['dataSendToFather'],
  template: `<div>Father:Received {{dataSendToFather}}</div>`
})

var demo = new Vue({
  el: '#demo'
})
```
父组件显示：Father:Received dataFromChild

用途：当多个父组件复用同一个子组件时，可以设置backPath为路由返回路径来解决返回问题。

## 修改element样式问题
在使用element-ui组件时，难免会出现bug，而一般的CSS方法无法覆盖element自身样式，因为element使用PostCSS来作为CSS编译器从而加入Scope（作用域）概念，这样样式不会冲突。
```css
// 编译前
.container {
  width: 100%
}
// 编译后
.container[_a12fq] {
  width: 100%
}
```
这样我们只能在样式的父级添加一个class来覆盖样式，具体实际操作如下

发现输入框宽度不对称

![element0.png](/assets/FrontEnd/Component/element/element0.png)

在App.vue写CSS，将CSS代码挂载到所有页面
```css
<style lang="scss">
.operation-container {
	.el-input {
		width: 100% !important;
	}
	.el-select {
		width: 100% !important;
	}
}
</style>
```

找到输入框的父级class覆盖
```html
<el-form class="operation-container" :model="param" :label-position="labelPosition" ref="param" label-width="40%">
<el-col :span="8">
    <el-form-item label="贷款人姓名" >
        <el-input v-model="param.realName" size="small"></el-input>
    </el-form-item>
</el-col>
......
```

修复bug完成

![element1.png](/assets/FrontEnd/Component/element/element1.png)