1. v-html 会有XSS风险，会覆盖子组件
2. computed 有缓存 data不变则不会重新计算，如果 v-model中使用了computed中定义的属性，computed属性要定义get和set
3. watch要进行深度监听，监听复杂数据类型的吧变化，属性里要定义`handler`函数和`deep：true`属性
4. watch 监听引用类型，拿不到oldVal，因为指针相同
5. class 和 style 的使用

```vue
  <p :class="{black:isBlack,yellow:isYellow}"></p>
  <p :class="[black,yellow]"></p>
  <p :style="styleData"></p>
  // isBlack,isYellow是布尔类型值
  // black，yellow属性定义了类名的变量
  // styleData是一个对象类型的值，使用驼峰命名
```

6. v-if 和 v-show 的区别

​	使用`v-if`，如果不该显示就不会渲染dom，使用`v-show`不该显示也会渲染dom，然后`display:none`

7. v-for遍历对象

```vue
<li v-for="(item,key,index) in obj">
  {{index}} - {{key}} - {{item.title}}
</li>
obj:{
  a:{title:"标题1"},
  b:{title:"标题2"},
  c:{title:"标题3"}
}
```

8. 使用`v-for`时，key尽量不要是`random`或`index`
9. `v-for`和`v-if`不能一起使用（不能在同一个标签上），`v-for`的优先级高于`v-if`
10. 事件的event参数

```vue
@click="increment1" // 可以直接获取event
@click="increment2(2,$event)" // 需要再自定义参数后面进行传递
increment1(event) {
	...
}
increment2(val,event) {
	...
}
// event 是原生的，事件被挂载到当前元素 event.target/event.currentTarget
// 事件修饰符:stop、prevent、capture、self
// 按键修饰符:@click.ctrl(即使 alt 或者 shift 被一同按下时也会触发)，@click.ctrl.exact(有且只有 ctrl 被按下时才触发)，@click.exact(没有任何系统修饰符被按下的时候才触发)
// 表单修饰符：lazy、number(转换成数字)、trim(去空格)
```

11. 父子组件生命周期

- 初次渲染时 生命周期钩子执行顺序是： 父组件 created、子组件 created、子组件 mounted、父组件 mounted
-  更新（状态）时生命周期钩子执行顺序是：父组件 beforeUpdate、子组件 beforeUpdate、子组件 updated、父组件 updated
- 销毁阶段生命周期钩子执行顺序是：父组件 beforeDestroy、子组件 beforeDestroy、子组件 destroyed、父组件 destroyed

12. 组件通信
    - `props`和`$emit`
    - `event bus`
13. 自定义v-model

```vue
<!-- index.vue -->
<p>{{name}}</p>
<CustomVModel v-model="customVmodel"></CustomVModel>

<!-- customVmodel 对应props中的text，能做到子组件修改父组件中定义的值 -->
<!-- CustomVModel.vue -->
<input type="text" :value="text" @change="$emit('change',$event.target.value)" />
export default {
  model:{
    prop:"text", //对应props中的text属性
    event:"change"
  }
  props:{
    text:String,
    default(){
      return ''
    }
  }
}
```

14. `$nextTick`

​		vue 是异步渲染的，data 改变之后，DOM 不会立刻渲染，`\$nextTick` 会在 DOM 渲染之后被触发，以获取最新 DOM 节点。 例如 data 中定义了一个列表 list，给一个 button 添加点击方法，每次点击给 list 添加三个元素之后打印出 list 的长度，如果不用`\$nextTick`，打印出的长度会是旧的 list 的长度。`$nextTick`是异步渲染，待 dom 渲染完再回调；页面渲染时会将 data 的修改做整合，多次 data 修改只会渲染一次。这样的设计满足性能需求和执行效率。

15. slot插槽

- 基本使用

```vue
<!-- slotDemo.vue -->
<template>
  <a :href="url">
    <slot> 默认内容，即父组件没设置内容时，这里显示. </slot>
  </a>
</template>
export default {
	props: ['url'],
  data(){
    return {}
  }
}

<!-- index.vue -->
<slotDemo :url="website.url">
  {{website.title}} // 这里插入标签或者自定义组件等也都可以
</slotDemo>
```

- 作用域插槽

```vue
<!-- scopedSlotDemo -->
<template>
	<a :href="url">
    <slot :slotData="website">
      {{website.subTitle}} // 默认值显示subTitle，即父组件不传值时默认显示的值
    </slot>
  </a>
</template>
export default {
	props: ['url'],
  data() {
    return {
      website:{
      url:'http://wangEditor.com/',
      title:'wangEditor',
      subTitle:'轻量级富文本编辑器'
      }
    }
  }
}


<!-- index.vue -->
<scopedSlotDemo :url="website.url">
  <template v-slot="slotProps"> // slotProps这个名称可自己定义
		{{slotProps.slotData.title}} // slotData对应子组件slot标签上定义的属性
  </template>
</scopedSlotDemo>
```



- 具名插槽

```vue
在 2.6.0 中，我们为具名插槽和作用域插槽引入了一个新的统一的语法 (即 v-slot 指令)。它取代了 slot 和 slot-scope 这两个目前已被废弃但未被移除且仍在文档中的 attribute。
<!-- NamedSlot组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>


<!-- index.vue -->
<NamedSlot>
  <template v-slot:header>
<h1>将插入到header slot中</h1>
  </template>
  <p>将插入到main中未命名的slot中</p>
  <template v-slot:footer>
<h1>将插入到footer slot中</h1>
  </template>
</NamedSlot>
```

16. 动态、异步组件

- 动态组件

```vue
 <component v-bind:is="currentTabComponent"></component>
// 动态组件应用场景：如新闻详情页；内容从上到下排布有：text 组件、text 组件、image 组件、text 组件、video 组件...
```

- 异步组件

```vue
// 组件什么时候使用什么时候再加载，使用方法就是不使用传统引入组件的方式，而是以下面这种方式来引用
components:{
	FormDemo:()=>import("../BaseUse/FormDemo")
}
```



17. keep-alive 用于缓存组件，应用场景如需要频繁切换不需要重复渲染的 tab 组件。
18. mixin   

多个组件有相同的逻辑，使用 `mixin` 抽离出来，`mixin` 并不是完美的解决方案，会有一些问题：(1.变量来源不明确，不利于阅读；2.多个 `mixin` 可能会造成命名冲突；3.`mixin` 和组件可能出现多对多的关系，复杂度耦合度较高)；`vue3` 提出的 `CompositionAPI` 旨在解决这些问题...)

19. vuex
20. Vue Router

