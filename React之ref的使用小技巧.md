1.  Ref 获取真实 DOM 元素和获取类组件实例

2. React 内部如何处理Ref

3.  Ref 对象的创建

   所谓 Ref 对象的创建，就是通过 React.createRef 或者 React.useRef 来创建一个 Ref 原始对象，一个标准的 ref 对象应该是如下的样子：

   ```js
   {
       current:null , // current指向ref对象获取到的实际内容，可以是dom元素，组件实例，或者其他。
   }
   ```

   ```react
   class Index extends React.Component{
       constructor(props){
          super(props)
          this.currentDom = React.createRef(null)
       }
       componentDidMount(){
           console.log(this.currentDom)
       }
       render= () => <div ref={ this.currentDom } >ref对象模式获取元素或组件</div>
   }
   
   React.createRef 的底层逻辑:
   // react/src/ReactCreateRef.js
   export function createRef() {
     const refObject = {
       current: null,
     }
     return refObject;
   }
   // createRef 只做了一件事，就是创建了一个对象，对象上的 current 属性，用于保存通过 ref 获取的 DOM 元素，组件实例等。 createRef 一般用于类组件创建 Ref 对象，可以将 Ref 对象绑定在类组件实例上，这样更方便后续操作 Ref。
   // 注意：不要在函数组件中使用 createRef，否则会造成 Ref 对象内容丢失等情况。
   ```

   类组件获取 Ref 三种方式

   1.  Ref属性是一个字符串

      ```react
      /* 类组件 */
      class Children extends Component{  
          render=()=><div>hello,world</div>
      }
      /* TODO:  Ref属性是一个字符串 */
      export default class Index extends React.Component{
          componentDidMount(){
             console.log(this.refs)
          }
          render=()=> <div>
              <div ref="currentDom"  >字符串模式获取元素或组件</div>
              <Children ref="currentComInstance"  />
          </div>
      }
      // 用一个字符串 ref 标记一个 DOM 元素，一个类组件(函数组件没有实例，不能被 Ref 标记)。React 在底层逻辑，会判断类型，如果是 DOM 元素，会把真实 DOM 绑定在组件 this.refs (组件实例下的 refs )属性上，如果是类组件，会把子组件的实例绑定在 this.refs 上。
      ```

      

   2. Ref 属性是一个函数

      ```react
      class Children extends React.Component{  
          render=()=><div>hello,world</div>
      }
      /* TODO: Ref属性是一个函数 */
      export default class Index extends React.Component{
          currentDom = null
          currentComponentInstance = null
          componentDidMount(){
              console.log(this.currentDom)
              console.log(this.currentComponentInstance)
          }
          render=()=> <div>
              <div ref={(node)=> this.currentDom = node }  >Ref模式获取元素或组件</div>
              <Children ref={(node) => this.currentComponentInstance = node  }  />
          </div>
      }
      // 用一个函数来标记 Ref 的时候，将作为 callback 形式，等到真实 DOM 创建阶段，执行 callback ，获取的 DOM 元素或组件实例，将以回调函数第一个参数形式传入，所以可以像上述代码片段中，用组件实例下的属性 currentDom和 currentComponentInstance 来接收真实 DOM 和组件实例。
      ```

      

   3. Ref属性是一个ref对象

      ```react
      class Children extends React.Component{  
          render=()=><div>hello,world</div>
      }
      export default class Index extends React.Component{
          currentDom = React.createRef(null)
          currentComponentInstance = React.createRef(null)
          componentDidMount(){
              console.log(this.currentDom)
              console.log(this.currentComponentInstance)
          }
          render=()=> <div>
               <div ref={ this.currentDom }  >Ref对象模式获取元素或组件</div>
              <Children ref={ this.currentComponentInstance }  />
         </div>
      }
      ```

      

   函数组件创建与获取Ref

   ```react
   export default function Index(){
       const currentDom = React.useRef(null)
       React.useEffect(()=>{
           console.log( currentDom.current ) // div
       },[])
       return  <div ref={ currentDom } >ref对象模式获取元素或组件</div>
   }
   ```

   

4. React 本身对Ref的处理

   主要指的是对于标签中 ref 属性，React 是如何处理以及 React如何 转发 Ref 

## Ref高阶用法

1.  forwardRef 转发 Ref, forwardRef 接收父级元素标记的 ref 信息，并把它转发下去，使得子组件可以通过 props 来接受到上一层级或者是更上层级的ref

   场景一：跨层级获取到子子...元素

   比如想要通过标记子组件 ref ，来获取孙组件的某一 DOM 元素，或者是组件实例

   ```react
   // 孙组件
   function Son (props){
       const { grandRef } = props
       return <div>
           <div> i am alien </div>
           <span ref={grandRef} >这个是想要获取元素</span>
       </div>
   }
   // 父组件
   class Father extends React.Component{
       constructor(props){
           super(props)
       }
       render(){
           return <div>
               <Son grandRef={this.props.grandRef}  />
           </div>
       }
   }
   const NewFather = React.forwardRef((props,ref)=> <Father grandRef={ref}  {...props} />)
   ```

   场景二：合并转发ref