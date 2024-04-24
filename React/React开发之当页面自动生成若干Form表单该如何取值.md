# React开发之当页面自动生成若干Form表单该如何取值

> 在PC后台业务开发中有这样一个需求：

1. `动态`为一个服务`对象`添加若干`业务类型`
2. 每添加一个业务类型，相应的要填写一些`表单数据`以及此业务`所需配件方案`等
3. 已添加的业务类型可删除
4. 业务类型下添加的`所需配件方案`也可删除
5. 所需的若干业务类型都添加并填写数据完毕，最终统一保存

> 基本效果如下


> 用到的技术点：
1. `antd Form`组件基本API
2. jsx语法、数据驱动视图，map遍历渲染内容
3. 最终保存时如何得到  `根据数据自动生成` 的若干form表单的值，最初考虑的是给每一个遍历渲染的Form都定义一个ref,但查阅了react ref相关知识点，发现无法实现本需求（可能是我没想到）。于是决定借助antd form提供的 `onValuesChange`来实现。具体方式可以看下面的代码

## 基本代码

> js部分伪代码

```js
// 定义包含所有项目的数据集
// 基本结构为：
// [
//   {
//     title: data.work_time.name, // 项目的名称
//     id: data.work_time.id, // 项目名称对应的id 后端需要
//     formList: plan_list_form_list, // 用于渲染表单结构的数据集 结构如下定义：
//   },
//   ...
// ]
const [plan_list, set_plan_list, get_plan_list] = useGetState([])

// 定义包含所有form表单填写的数据的数据集
// 基本结构为：
// {
//   id1: {
//     name: '',
//     month: ''
//   },
//   id2: {
//     name: '',
//     month: '
//   },
//   ...
// }
const [ allFormData, setAllFormData, getAllFormData ] = useGetState({});

// 定义方案列表 schemeList： { [项目id]:[{方案名， 配件列表},{方案名，配件列表}] } 操作逻辑相通，这里不再赘述

const plan_list_form_list = [
  {
    name: 'name',
    label: 'testjkhj',
    type: 'input',
    rules: [
      { required: true },
      {
        pattern: /^([1-9]\d*|[0]{1,1})$/,
        message: '...',
      }
    ]
  },
  {
    name: 'month',
    label: 'testlala',
    type: 'input'
  },
  ....
]


// 点击新增项目按钮调用的方法
function onCreate() {
  // 往项目的数据集中加入一条数据
  set_plan_list([
    ...plan_list,
    {
      title: data.work_time.name,
      key: data.work_time.id,
      formList: [...plan_list_form_list], // 基本的表单结构
    }
  ])
  // 往表单的数据集中加入一条
  let currentFormData = {};
  currentFormData[data.work_time.value] = {
    mileage: '',
    month: '',
    oil_consumption: '',
    parts_standard: ''
  }
  setAllFormData({...allFormData, ...currentFormData})
}


// 当往每一个项目的表单数据填入内容时
// 触发当前表单的onValuesChange(changedValues, allValues)
function onFormValuesChange(changedValues, allValues,formKey) { 
// onFormValuesChange是传给子组件BasicForm的事件属性
// 当BasicForm中的onValuesChange触发，就会调用onFormValuesChange方法，并将相关参数传过来
// formKey绑定在每一个BasicForm中，取值为每一个项目集的key,用于区分不同项目的form表单

let allFormDataCopy = JSON.parse(JSON.stringify(allFormData));
allFormDataCopy[formKey] = allValues; // 获取并写入当前form的所有值
setAllFormData(allFormDataCopy)
}

// 最终保存
function onSave() {
  // 组合页面写入的项目集数据、表单集数据、方案集数据等为后端需要的格式
  // 调接口提交
  // ...
}


```
> jsx部分伪代码

```js
import { Card, Collapse, Form, ...} from 'antd'
function AddPlan () {
  return (
    <Card title="项目">
      <Collapse>
        {
          plan_list.map((item,index) => {
            return (
              <Panel extra={panelExtra(item, index)} header={item.title} key={(index+1).toString()} forceRender={true}>
                  // 如果从编辑操作进入这个页面，form表单中要渲染已存在的数据，拿到后端给的数据，并设置在js中initialValues
                  <BasicForm initialValues={item.initialValues} formKey={item.key} onFormValuesChange={onFormValuesChange} formList={item.formList} />    
                  <Space size="small" style={{display: 'flex', alignItems: 'center', marginBottom: '15px'}}>
                    <h3 style={{marginBottom: '0'}}>配件方案</h3>
                    <Button type="primary" size="small" onClick={()=>onAddScheme(item.key)}>添加配件方案</Button>
                  </Space>
                  {
                    (Object.keys(getSchemeList()).length !==0 && getSchemeList()[item.key]) && (
                      getSchemeList()[item.key].map((cpt, j)=>{
                        return(
                          <div key={j}>
                            <Space size="small" style={{display: 'flex', alignItems: 'center', marginBottom: '15px', marginTop: '15px'}}>
                              <h4 style={{marginBottom: '0'}}>
                                {cpt.program_name}
                              </h4>
                              <MinusCircleOutlined style={{ fontSize: '18px', color: '#f5222d' }} onClick={()=>onDeleteScheme(item.key, j)}/>
                            </Space>
                            <Table bordered pagination={false} columns={spareColumns} dataSource={cpt.spare_parts_list} rowKey="spare_parts_id"></Table>
                          </div>
                        )
                      })
                    )
                  }
              </Panel>
            )
          })
        }
      </Collapse>
    </Card>
  )
}
```
## 补充
> 受vue中ref使用方式的启发，设想react是否和vue一样，当组件绑定的ref传入函数时，函数的参数就是当前实例，于是在react项目中动手实践了一番，发现确如所想，如此一来，上面的逻辑就变得更简单一些了。具体操作如下代码

```js
// 主体代码、其它省略

function useRefs() {
  const refs = []
  const cache = []
  const setRefs = (index) => {
    if(!cache[index]) {
      cache[index] = (el) => {
        refs[index] = el;
      }
    }
    return cache[index]
  }
  return [refs, setRefs]
}

const [refs, setRefs] = useRefs()

console.log(refs, 'refs')

function onSave() {
  // 取所有表单数据
  if(refs.length) {
    refs.map(item=>{
      console.log(item.getFieldsValue())
    })
  }
}

<Card>
  <Collapse>
    {
      plan_list.map((item, index) => {
        return (
          <Panel>
            <BasicForm ref={setRefs(index)} />    
          </Panel>
        )
      })
    }
  </Collapse>
</Card>
```



