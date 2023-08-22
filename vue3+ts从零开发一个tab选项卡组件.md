# vue3+ts从零开发一个tab选项卡组件

## tab组件功能点如图

## 实现思路和大致代码

> 定义两个基本组件文件夹分别是：tab、tab-item

1. tab组件可配置属性为：

```js
import { makeNumberProp } from '../utils/props'

// export const makeNumberProp = <T>(defaultVal: T) => ({
//   type: Number,
//   default: defaultVal
// })

export const tabProps = {
  vertical: Boolean, // 设置标签栏模式为垂直， 默认水平
  active: makeNumberProp(0), // 设置默认选中第几项
  swipeThreshold: makeNumberProp(5) // 设置最多可展示多少个标签，超过时滚动
}
```

2. tab-item组件基本可配置属性为：

```js
export const tabItemProps = {
  title: String, // 标签名称
  disabled: Boolean // 标签是否禁用
}
```

> tab.tsx 基本功能点：

1. 设置默认选中index, 点击选项卡标签，标签变为激活状态，当前index更改为被点击项的index，内容随之显示
2. 

```js
import {
  defineComponent,
  onMounted,
  ref,
  getCurrentInstance,
  InjectionKey,
  ComputedRef,
  ExtractPropTypes,
  reactive,
  watch,
  nextTick,
  CSSProperties,
  computed
} from 'vue'
import { useNamespace } from '../hooks/use-namespace'
import { onMountedOrActivated } from '../hooks/on-mounted-or-activated'
import './tab.scss'
import { tabProps } from './props'
import { ComponentInstance, type Numeric } from '../utils/basic'
import { useRefs } from '../hooks/use-refs'

export type TabProps = ExtractPropTypes<typeof tabProps>

export type TabsProvide = {
  // id: string
  props: TabProps
  // setLine: () => void
  // onRendered: (name: Numeric, title?: string) => void
  // scrollIntoView: (immediate?: boolean) => void
  currentName: ComputedRef<Numeric | undefined>
}

import { useChildren } from '../hooks/use-relation'
import TabTitle from './tab-title'
export const TAB_KEY: InjectionKey<TabsProvide> = Symbol('so-tab')
const ns = useNamespace('tab')

export default defineComponent({
  name: 'SoTab',
  props: tabProps,
  emits: ['click-tab'],
  setup(props, { emit, slots }) {
    const { children, linkChildren } = useChildren(TAB_KEY)
    const [titleRefs, setTitleRefs] = useRefs<ComponentInstance>()

    const state = reactive({
      currentIndex: 0, //定义当前选中选项索引，默认为0 配合props.active进行设置
      lineStyle: {} as CSSProperties
    })
    // eslint-disable-next-line vue/return-in-computed-property
    // 定义currentName
    const currentName = computed(() => {
      const activeTab = children[state.currentIndex]
      if (activeTab) {
        return getTabName(activeTab, state.currentIndex)
      }
    })

    // 定义是否开启标签栏滚动
    const scrollable = computed(() => children.length > props.swipeThreshold)

    const getTabName = (tab: ComponentInstance, index: number): Numeric => {
      return tab.name ?? index
    }

    const setLine = () => {
      nextTick(() => {
        const titles = titleRefs.value
        if (titles && titles[state.currentIndex]) {
          const title = titles[state.currentIndex].$el
          const left = title.offsetLeft + title.offsetWidth / 2
          const top = title.offsetTop

          let lineStyle: CSSProperties = {}
          if (props.vertical) {
            lineStyle = {
              width: '3px',
              height: '26px',
              transform: `translateY(${top}px)`
            }
          } else {
            lineStyle = {
              width: '40px',
              height: '3px',
              transform: `translateX(${left}px) translateX(-50%)`
            }
          }

          state.lineStyle = lineStyle
        }
      })
    }

    const onClickTabItem = (
      item: ComponentInstance,
      index: number,
      event: MouseEvent
    ) => {
      // 点击tabItem 处理标签的样式
      setCurrentIndex(index)
      const { title } = item
      // 处理完以上之后触发外面传进来的click-tab事件并传递相关的一些参数以供使用者使用
      emit('click-tab', {
        index,
        title,
        event
      })
    }

    const setCurrentIndex = (index: number) => {
      state.currentIndex = index
    }

    const renderNav = () => {
      return children.map((item: ComponentInstance, index: number) => {
        return (
          <TabTitle
            isActive={state.currentIndex == index}
            title={item.title}
            v-slots={{ title: item.$slots.title }}
            scrollable={scrollable.value}
            disabled={item.disabled}
            ref={setTitleRefs(index)}
            onClick={(event: MouseEvent) => onClickTabItem(item, index, event)}
          />
        )
      })
    }
    const renderLine = () => {
      const classes = ns.e('line')
      return <div class={classes} style={state.lineStyle}></div>
    }

    const renderHeader = () => {
      const classes = {
        [ns.e('header')]: true,
        [ns.em('header', 'complete')]: scrollable.value
      }

      return (
        <div class={classes}>
          {renderNav()}
          {renderLine()}
        </div>
      )
    }

    const init = () => {
      if (props.active) {
        state.currentIndex = props.active
      }
      setLine()
    }

    watch(
      () => props.active,
      () => {
        setCurrentIndex(props.active)
      }
    )

    watch(
      () => children.length,
      () => {
        nextTick(() => {
          setLine()
        })
      }
    )

    watch(
      () => state.currentIndex,
      () => {
        setLine()
      }
    )

    linkChildren({ props, currentName })

    onMountedOrActivated(init)

    return () => {
      return (
        <div class={[ns.b(), props.vertical ? ns.m('vertical') : '']}>
          {renderHeader()}
          <div>{slots.default && slots.default()}</div>
        </div>
      )
    }
  }
})

```
> tab-item.tsx