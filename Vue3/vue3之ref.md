# vue3之ref的使用

## ref基本使用
> `ref()`接受一个内部值，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性`.value`
> ref 对象是可更改的，也就是说你可以为`.value`赋予新的值。它也是响应式的，即所有对 `.value`的操作都将被追踪，并且写操作会触发与之相关的副作用

> 示例
```js
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

## 和shallowRef()的区别
> `shallowRef()` 是 `ref()`的浅层作用形式，即内部值不会被深层递归地转为响应式。只有对 .value 的访问是响应式的。
> 示例
```js
const state = shallowRef({ count: 1 })
// 不会触发更改
state.value.count = 2
// 会触发更改
state.value = { count: 2 }


const refState = ref({count: 1})
// 会触发更改
refState.value = { count: 2 }
// 会触发更改
refState.value.count = 2
```

## 为ref标注类型

> 不定义类型时，ref会根据初始化时的值推导其类型

```js
import { ref } from 'vue'
// 推导出的类型：Ref<number>
const year = ref(2020)
// => TS Error: Type 'string' is not assignable to type 'number'.
year.value = '2020'
```

> 在调用ref()时，如果传入一个泛型参数，则会覆盖默认的类型推导

```js
// 得到的类型：Ref<string | number>
const year = ref<string | number>('2020')

year.value = 2020 // 成功！
```

> 或者可以通过使用 Ref 这个类型：

```js
import { ref } from 'vue'
import type { Ref } from 'vue'

const year: Ref<string | number> = ref('2020')

year.value = 2020 // 成功！
```

> 如果你指定了一个泛型参数但没有给出初始值，那么最后得到的就将是一个包含 undefined 的联合类型：

```js
// 推导得到的类型：Ref<number | undefined>
const n = ref<number>()
```

## 复杂业务场景中ref的使用

> 有时我们可能需要在一个父组件中渲染多个同一子组件，在遍历渲染子组件的同时每个子组件都需要绑定一个ref属性

> 思路
1. 在遍历中通过index作为参数调用ref
2. 声明一个用来存储所有子元素ref的数组refs
3. 每次调用都将根据index生成的元素实例 存储到数组refs中

> 写一个具备以上功能的hook`useRefs`
```js
// use-refs.ts
import { ref, Ref, onBeforeUpdata } from 'vue'
export function useRefs<T = Element>() {
  // refs存所有的子组件的ref
  const refs = ref([]) as Ref<T[]>
  // cache存的是一堆方法的引用，参数是vue组件实例
  const cache: Array<(el: unknown) => void> = []

  onBeforeUpdate(()=>{
    refs.value = [] //组件更新时清空refs
  })

  // 定义setRefs方法，返回的是一个函数，在组件中调用setRefs等于调用这个返回的函数
  const setRefs = (index: number) => {
    // 如果当前index的子元素不存在
    if(!cache[index]) {
      cache[index] = (el: unknown) => {
        refs.value[index] = el as T;
      }
    }
    return cache[index]
  }

  return [refs, setRefs] as const
}
```
> 在业务开发中的使用,以下为总体伪代码，实际根据业务需求进行调整

```js
import { useRefs } from '../hooks/use-refs'
const [childRefs, setChildRefs] = useRefs<ComponentInstance>()
export default defineComponent({
  setup(props,ctx){
    return ()=>{
      return (
        <div>
          // 渲染多个子元素
          {[...].map((item, index)=>{
            return (
              <Child
                ref={setChildRefs(index)}
                // ref={el => getEl(el)} // 当ref等于一个函数时，这个函数的参数是当前组件实例
                onClick={(event: MouseEvent) => onClick(item, index, event)}
              />
            )
          })}
          <div>other content...</div>
        </div>
      )
    }
  }
})
```