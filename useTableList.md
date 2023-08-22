# 封装数据列表的操作

> 在使用react开发后台管理系统时，table数据列表的需求非常常见。每个数据列表都包含有table展示、table分页、表单搜索、功能按钮操作等功能。
> 如果每个页面都写一遍这些功能，代码量将非常巨大且繁琐
> 为了能方便快捷的完成数据表相关的操作，将更多的经历放在更重要的业务开发中，我考虑写一个hook,通过一个hook完成以上罗列的功能。

## 我的业务场景如下：
1. 使用的UI组件是antd
2. 每个数据展示页面基本都包含搜索、展示、操作
3. 能够缓存数据列表页面的操作

## 借助useAntdTable
> 因为useAntdTable封装了常用的 antd Form 与 antd Table 联动逻辑，所以借助useAntdTable可以快速实现我在业务中需要开发的功能。
> 关于useAntdTable可查看官网![useAntdTable](https://ahooks.js.org/zh-CN/hooks/use-antd-table)

## 核心代码

### useTableListService.js
方法概要：
1. service 是获取数据的接口
2. 因为要与搜索表单联动 所以还需要每个页面的搜索表单实例
3. 如果改页面需要缓存操作记录 那么还需要提供一个cacheKey参数
4. 其它参数的含义可以参看 ahooks 官网的 useAntdTable

```js
import { useAntdTable } from 'ahooks';

export function useTableListService(service, httpParams={}) {
  const { form, cacheKey } = httpParams;
  // 调用useAntdTable, 通过传入的API方法、搜索表单实例、table渲染参数，得到一些开发中需要的返回值
  let { tableProps, search, params, data } = useAntdTable(service, {
    form,
    defaultPageSize: defaultPageSize || 20,
    debounceWait: 300,
    cacheKey
  });
  // 表格的一些配置设置，具体设置与含义可参看antd的Table组件API
  tableProps.pagination.showQuickJumper = true;
  tableProps.pagination.showTotal = total => `共 ${ total } 条`;
  tableProps.pagination.pageSizeOptions = [20, 50, 100];
  tableProps.pagination.showSizeChanger = true;

  // 最后将数据列表页面基本需要的参数返回
  // params是表单相关的参数
  // data是调接口获取的数据
  // tableProps是antdTable需要的配置参数，用于在页面中传给Table组件，表格相关的分页、跳转等方法已被封装在其中，不再需要针对分页点击跳转，更改pagesize等操作做处理
  // search中包含了比较常用的方法 submit和reset //分别用来提交表单和重置当前表单
  return { tableProps, search, params, data}
}
```

### 引入useTableListService并使用

```js
import { Form } from 'antd'

function List () {

  // getList即useAntdTable 中的 service
  // 接收两个参数，第一个参数为分页数据 { current, pageSize, sorter, filters, extra }，第二个参数为表单数据
  // service 返回的数据结构为 { total: number, list: Item[] }
  function getList({current, pageSize}, formData) {
    // 拿到的表单和分页参数如果需要处理后再传给后端，在这里进行
    const params = {
      page: current, // 我这边后端需要的分页参数是 page和rows
      rows: pageSize,
      ...formData // 表单中的数据如果比较复杂需要处理的也可以先处理再传值
    }
    // 参数处理完 调用接口获取数据列表并将取到的total和list返回
    return httpGetCustomList(params).then(res=>({
      total: res.data.total,
      list: res.data.list,
    }))
  }

  const { tableProps, search, params, data } = useTableListService(getList, {form: searchForm, cacheKey: 'cache-key'})
  const {submit, reset} = search // 搜索表单的搜索和重置方法，传给封装的SearchForm组件使用
  const [ form ] = Form.useForm()
  // searchFormList是用于渲染搜索表单结构的数据
  const searchFormList = [
    {label: '', type: 'input/select/cascader...', name: 'username'},
    {label: '', type: 'input/select/cascader...', name: 'contact'}
  ]
  const columns = [] //表格数据渲染 同antd table的columns
  const cardExtra = () => (<Button onClick={onCreate}>新建</Button>) // cardExtra用于返回渲染表格顶部的若干操作按钮
  const onCreate = () => {}
  return (
    <>
      // SearchForm是基于项目需求封装的组件，用于快捷实现搜索表单需求
      <SearchForm form={form} formList={searchFormList} search={submit} reset={reset}></SearchForm>
      // BasicTable是基于项目需求、基于antd Table二次封装的组件
      <BasicTable columns={columns} cardExtra={cardExtra} tableProps={tableProps} rowKey="id"></BasicTable>
    </>
  )
}

export default List
```
### 最终效果

> 以上，如有问题，欢迎指正。
