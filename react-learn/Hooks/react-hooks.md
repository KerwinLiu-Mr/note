# Hook
- Hook 是什么？ Hook 是一个特殊的函数，**它可以让你“钩入” React 的特性**。例如，useState 是允许你在 React 函数组件中添加 state 的 Hook。
- Hook 在 class 内部是不起作用的。但你可以使用它们来取代 class 。
~~~js
const Example = (props) => {
  // 你可以在这使用 Hook
  return <div />;
}
~~~
- 之前可能把它们叫做“无状态组件”。但现在我们为它们引入了使用 React state 的能力，所以我们更喜欢叫它”函数组件”。
- 什么时候我会用 Hook？ 如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它转化为 class。现在你可以在现有的函数组件中使用 Hook。
>注意：
在组件中有些特殊的规则，规定什么地方能使用 Hook，什么地方不能使用。我们将在 Hook 规则中学习它们。
>
## 为什么使用Hook

### 组件复用困局

- 组件并不是单纯的信息孤岛，组件之间是可能会产生联系的，一方面是数据的共享，另一个是功能的复用：

1. 对于组件之间的数据共享问题，React官方采用单向数据流（Flux）来解决

   对于（有状态）组件的复用，React团队给出过许多的方案，早期使用CreateClass + Mixins，在使用Class Component取代CreateClass之后又设计了`Render Props`和`Higher Order Component`，直到再后来的Function Component+ Hooks设计，React团队对于组件复用的探索一直没有停止
- HOC使用（老生常谈）的问题：
  >
    嵌套地狱，每一次HOC调用都会产生一个组件实例
    可以使用类装饰器缓解组件嵌套带来的可维护性问题，但装饰器本质上还是HOC
    包裹太多层级之后，可能会带来props属性的覆盖问题
  >
- Render Props：
  >
  数据流向更直观了，子孙组件可以很明确地看到数据来源
  但本质上Render Props是基于闭包实现的，大量地用于组件的复用将不可避免地引入了callback hell问题
  丢失了组件的上下文，因此没有this.props属性，不能像HOC那样访问this.props.children
  >
1. react-hooks可以让我们的代码的逻辑性更强，可以抽离公共的方法，公共组件。
2. react-hooks思想更趋近于函数式编程。用函数声明方式代替class声明方式，虽说class也是es6构造函数语法糖，但是react-hooks写起来更有函数即组件，无疑也提高代码的开发效率（无需像class声明组件那样写声明周期，写生命周期render函数等）
3. react-hooks可能把庞大的class组件，化整为零成很多小组件，useMemo等方法让组件或者变量制定一个适合自己的独立的渲染空间，一定程度上可以提高性能，减少渲染次数。这里值得一提的是，如果把负责 请求是数据 ➡️  视图更新的渲染组件，用react-hooks编写的话 ，配合immutable等优秀的开源库，会有更棒的效果(这里特别注意的是⚠️，如果乱用hooks，不但不会提升性能，反而会影响性能，带来各种各样的想不到的问题)。
## Hook使用规则
- Hook 就是 JavaScript 函数，但是使用它们会有两个额外的规则：
1. 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层以及任何 return 之前调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。
2. 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。（还有一个地方可以调用 Hook —— 就是自定义的 Hook 中，我们稍后会学习到。）
### 规则说明
- 我们可以在单个组件中使用多个 State Hook 或 Effect Hook
~~~js
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
~~~
- 那么 React 怎么知道哪个 state 对应哪个 useState？答案是 React 靠的是 Hook 调用的顺序。因为我们的示例中，Hook 的调用顺序在每次渲染中都是相同的，所以它能够正常工作：
~~~js
// ------------
// 首次渲染
// ------------
useState('Mary')           // 1. 使用 'Mary' 初始化变量名为 name 的 state
useEffect(persistForm)     // 2. 添加 effect 以保存 form 操作
useState('Poppins')        // 3. 使用 'Poppins' 初始化变量名为 surname 的 state
useEffect(updateTitle)     // 4. 添加 effect 以更新标题

// -------------
// 二次渲染
// -------------
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
useEffect(persistForm)     // 2. 替换保存 form 的 effect
useState('Poppins')        // 3. 读取变量名为 surname 的 state（参数被忽略）
useEffect(updateTitle)     // 4. 替换更新标题的 effect

// ...
~~~
- 只要 Hook 的调用顺序在多次渲染之间保持一致，React 就能正确地将内部 state 和对应的 Hook 进行关联。但如果我们将一个 Hook (例如 persistForm effect) 调用放到一个条件语句中会发生什么呢
~~~js
 // 🔴 在条件语句中使用 Hook 违反第一条规则
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
~~~
- 在第一次渲染中 name !== '' 这个条件值为 true，所以我们会执行这个 Hook。但是下一次渲染时我们可能清空了表单，表达式值变为 false。此时的渲染会跳过该 Hook，Hook 的调用顺序发生了改变：
~~~js
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
// useEffect(persistForm)  // 🔴 此 Hook 被忽略！
useState('Poppins')        // 🔴 2 （之前为 3）。读取变量名为 surname 的 state 失败
useEffect(updateTitle)     // 🔴 3 （之前为 4）。替换更新标题的 effect 失败
~~~
- React 不知道第二个 useState 的 Hook 应该返回什么。React 会以为在该组件中第二个 Hook 的调用像上次的渲染一样，对应的是 persistForm 的 effect，但并非如此。从这里开始，后面的 Hook 调用都被提前执行，导致 bug 的产生。
- 这就是为什么 Hook 需要在我们组件的最顶层调用。如果我们想要有条件地执行一个 effect，可以将判断放到 Hook 的内部：
~~~js
 useEffect(function persistForm() {
    // 👍 将条件判断放置在 effect 中
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
~~~
## 使用 State Hook
### 声明 State 变量
- useState出现，使得react无状态组件能够像有状态组件一样，可以拥有自己state,useState的参数可以是一个具体的值，也可以是一个函数用于判断复杂的逻辑，函数返回作为初始值，usestate 返回一个数组，数组第一项用于读取此时的state值 ，第二项为派发数据更新，组件渲染的函数，函数的参数即是需要更新的值。useState和useReduce 作为能够触发组件重新渲染的hooks,**我们在使用useState的时候要特别注意的是，useState派发更新函数的执行，就会让整个function组件从头到尾执行一次，所以需要配合useMemo，usecallback等api配合使用，这就是我说的为什么滥用hooks会带来负作用的原因之一了**。
在 class 中，我们通过在构造函数中设置 this.state 为 { count: 0 } 来初始化 count state 为 0：
~~~js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
~~~
- 在函数组件中，我们没有 this，所以我们不能分配或读取 this.state。我们直接在组件中调用 useState Hook：
~~~js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量
  const [count, setCount] = useState(0);
~~~
- 调用 useState 方法的时候做了什么? 它定义一个 “state 变量”。我们的变量叫 count， 但是我们可以叫他任何名字，比如 banana。这是一种在函数调用时保存变量的方式 —— useState 是一种新方法，它与 class 里面的 this.state 提供的功能完全相同。一般来说，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留。
- useState 需要哪些参数？ useState() 方法里面唯一的参数就是初始 state。不同于 class 的是，我们可以按照需要使用数字或字符串对其进行赋值，而不一定是对象。在示例中，只需使用数字来记录用户点击次数，所以我们传了 0 作为变量的初始 state。（如果我们想要在 state 中存储两个不同的变量，只需调用 useState() 两次即可。）
- useState 方法的返回值是什么？ 返回值为：当前 state 以及更新 state 的函数。这就是我们写 const [count, setCount] = useState() 的原因。这与 class 里面 this.state.count 和 this.setState 类似，唯一区别就是你需要成对的获取它们。
>注意
你可能想知道：为什么叫 useState 而不叫 createState?
“Create” 可能不是很准确，因为 state 只在组件首次渲染的时候被创建。在下一次重新渲染时，useState 返回给我们当前的 state。否则它就不是 “state”了！这也是 Hook 的名字总是以 use 开头的一个原因。我们将在后面的 Hook 规则中了解原因。
~~~js
const DemoState = (props) => {
   /* number为此时state读取值 ，setNumber为派发更新的函数 */
   let [number, setNumber] = useState(0) /* 0为初始值 */
   return (<div>
       <span>{ number }</span>
       <button onClick={ ()=> {
         setNumber(number+1) /* 写法一 */
         setNumber(number=>number + 1 ) /* 写法二 */
         console.log(number) /* 这里的number是不能够即时改变的  */
       } } >num++</button>
   </div>)
}
~~~
- 不像 class 中的 this.setState，**更新 state 变量总是替换它而不是合并它**。
## Effect Hook
- 如果你想在function组件中，当组件完成挂载，dom渲染完成，做一些操纵dom,请求数据，那么useEffect是一个不二选择，如果我们需要在组件初次渲染的时候请求数据，那么useEffect可以充当class组件中的 componentDidMount , **但是特别注意的是，如果不给useEffect执行加入限定条件，函数组件每一次更新都会触发effect ,那么也就说明每一次state更新，或是props的更新都会触发useEffect执行，此时的effect又充当了componentDidUpdate和componentwillreceiveprops，所以说合理的用于useEffect就要给effect加入限定执行的条件，也就是useEffect的第二个参数，这里说是限定条件，也可以说是上一次useeffect更新收集的某些记录数据变化的记忆，在新的一轮更新，useeffect会拿出之前的记忆值和当前值做对比，如果发生了变化就执行新的一轮useEffect的副作用函数，useEffect第二个参数是一个数组，用来收集多个限制条件** 。
- 如果我们需要在组件销毁的阶段，做一些取消dom监听，清除定时器等操作，那么我们可以在useEffect函数第一个参数，结尾返回一个函数，用于清除这些副作用。相当与componentWillUnmount。
- 你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。
- useEffect 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。（我们会在使用 Effect Hook 里展示对比 useEffect 和这些方法的例子。）
~~~js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
~~~
- 当你调用 useEffect 时，就是在告诉 React 在完成对 DOM 的更改后运行你的“副作用”函数。由于副作用函数是在组件内声明的，所以它们可以访问到组件的 props 和 state。默认情况下，React 会在每次渲染后调用副作用函数 —— 包括第一次渲染的时候。（我们会在使用 Effect Hook 中跟 class 组件的生命周期方法做更详细的对比。）
- useEffect可以弥补函数组件没有生命周期的缺点。我们可以在useEffect第一个参数回调函数中，做一些请求数据，事件监听等操作，第二个参数作为dep依赖项，当依赖项发生变化，重新执行第一个函数。
### useEffect可以用作数据交互
~~~js
/* 模拟数据交互 */
function getUserInfo(a){
    return new Promise((resolve)=>{
        setTimeout(()=>{ 
           resolve({
               name:a,
               age:16,
           }) 
        },500)
    })
}
const DemoEffect = ({ a }) => {
    const [ userMessage , setUserMessage ] :any= useState({})
    const div= useRef()
    const [number, setNumber] = useState(0)
    /* 模拟事件监听处理函数 */
    const handleResize =()=>{}
    /* useEffect使用 ，这里如果不加限制 ，会是函数重复执行，陷入死循环*/
    useEffect(()=>{
        /* 请求数据 */
       getUserInfo(a).then(res=>{
           setUserMessage(res)
       })
       /* 操作dom  */
       console.log(div.current) /* div */
       /* 事件监听等 */
        window.addEventListener('resize', handleResize)
    /* 只有当props->a和state->number改变的时候 ,useEffect副作用函数重新执行 ，如果此时数组为空[]，证明函数只有在初始化的时候执行一次相当于componentDidMount */
    },[ a ,number ])
    return (<div ref={div} >
        <span>{ userMessage.name }</span>
        <span>{ userMessage.age }</span>
        <div onClick={ ()=> setNumber(1) } >{ number }</div>
    </div>)
}
~~~
### useEffect可以用作事件监听，还有一些基于dom的操作。
- useEffect可以用作事件监听，还有一些基于dom的操作。,别忘了在useEffect第一个参数回调函数，返一个函数用于清除事件监听等操作。
~~~js
const DemoEffect = ({ a }) => {
    /* 模拟事件监听处理函数 */
    const handleResize =()=>{}
    useEffect(()=>{
       /* 定时器 延时器等 */
       const timer = setInterval(()=>console.log(666),1000)
       /* 事件监听 */
       window.addEventListener('resize', handleResize)
       /* 此函数用于清除副作用 */
       return function(){
           clearInterval(timer) 
           window.removeEventListener('resize', handleResize)
       }
    },[ a ])
    return (<div  >
    </div>)
}
~~~
- useEffect 做了什么？ 通过使用这个 Hook，你可以告诉 React 组件需要在渲染后执行某些操作。React 会保存你传递的函数（我们将它称之为 “effect”），并且在执行 DOM 更新之后调用它。在这个 effect 中，我们设置了 document 的 title 属性，不过我们也可以执行数据获取或调用其他命令式的 API。
- 为什么在组件内部调用 useEffect？ 将 useEffect 放在组件内部让我们可以在 effect 中直接访问 count state 变量（或其他 props）。我们不需要特殊的 API 来读取它 —— 它已经保存在函数作用域中。Hook 使用了 JavaScript 的闭包机制，而不用在 JavaScript 已经提供了解决方案的情况下，还引入特定的 React API。
- useEffect 会在每次渲染后都执行吗？ 是的，默认情况下，它在第一次渲染之后和每次更新之后都会执行。（我们稍后会谈到如何控制它。）你可能会更容易接受 effect 发生在“渲染之后”这种概念，不用再去考虑“挂载”还是“更新”。**React 保证了每次运行 effect 的同时，DOM 都已经更新完毕**。

- 为什么要在 effect 中返回一个函数？ 这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。
- React 何时清除 effect？ React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React 会在执行当前 effect 之前对上一个 effect 进行清除。我们稍后将讨论为什么这将助于避免 bug以及如何在遇到性能问题时跳过此行为。
- 我们介绍了一个用于显示好友是否在线的 FriendStatus 组件。从 class 中 props 读取 friend.id，然后在组件挂载后订阅好友的状态，并在卸载组件的时候取消订阅：
~~~js
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
~~~
- **但是当组件已经显示在屏幕上时，friend prop 发生变化时会发生什么？** 我们的组件将继续展示原来的好友状态。这是一个 bug。而且我们还会因为取消订阅时使用错误的好友 ID 导致内存泄露或崩溃的问题。
- 在 class 组件中，我们需要添加 componentDidUpdate 来解决这个问题：
~~~js
 componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    // 取消订阅之前的 friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // 订阅新的 friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
~~~
- 忘记正确地处理 componentDidUpdate 是 React 应用中常见的 bug 来源。
- 使用 Hook 不会受到此 bug 影响, 因为 useEffect 默认就会处理。它会在调用一个新的 effect 之前对前一个 effect 进行清理
## useMemo
- useMemo接受两个参数，第一个参数是一个函数，返回值用于产生保存值。 第二个参数是一个数组，作为dep依赖项，数组里面的依赖项发生变化，重新执行第一个函数，产生新的值。
- 把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。
- 记住，传入 useMemo 的函数会在渲染期间执行。**请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴**，而不是 useMemo。
- 如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。
- 你可以把 useMemo 作为性能优化的手段，但不要把它当成语义上的保证。将来，React 可能会选择“遗忘”以前的一些 memoized 值，并在下次渲染时重新计算它们，比如为离屏组件释放内存。先编写在没有 useMemo 的情况下也可以执行的代码 —— 之后再在你的代码中添加 useMemo，以达到优化性能的目的。
### 缓存一些值，避免重新执行上下文
~~~js
const number = useMemo(()=>{
    /** ....大量的逻辑运算 **/
   return number
},[ props.number ]) // 只有 props.number 改变的时候，重新计算number的值。
~~~
### 减少不必要的dom循环
~~~js
/* 用 useMemo包裹的list可以限定当且仅当list改变的时候才更新此list，这样就可以避免selectList重新循环 */
 {useMemo(() => (
      <div>{
          selectList.map((i, v) => (
              <span
                  className={style.listSpan}
                  key={v} >
                  {i.patentName} 
              </span>
          ))}
      </div>
), [selectList])}
~~~
### 减少子组件渲染
~~~js
/* 只有当props中，list列表改变的时候，子组件才渲染 */
const  goodListChild = useMemo(()=> <GoodList list={ props.list } /> ,[ props.list ])
~~~
## useLayoutEffect 渲染更新之前的 useEffect
- useEffect 执行顺序 组件更新挂载完成 -> 浏览器dom 绘制完成 -> 执行useEffect回调 。
- useLayoutEffect 执行顺序 组件更新挂载完成 ->  执行useLayoutEffect回调-> 浏览器dom 绘制完成
- 所以说useLayoutEffect 代码可能会阻塞浏览器的绘制  如果我们在useEffect 重新请求数据，渲染视图过程中，肯定会造成画面闪动的效果,而如果用useLayoutEffect ，回调函数的代码就会阻塞浏览器绘制，- 所以可定会引起画面卡顿等效果，那么具体要用 useLayoutEffect 还是 useEffect ，要看实际项目的情况，大部分的情况 useEffect 都可以满足的。
~~~js
const DemoUseLayoutEffect = () => {
    const target = useRef()
    useLayoutEffect(() => {
        /*我们需要在dom绘制之前，移动dom到制定位置*/
        const { x ,y } = getPositon() /* 获取要移动的 x,y坐标 */
        animate(target.current,{ x,y })
    }, []);
    return (
        <div >
            <span ref={ target } className="animate"></span>
        </div>
    )
}
~~~
## useRef 获取元素 ,缓存数据
- 和传统的class组件ref一样，react-hooks 也提供获取元素方法 useRef,它有一个参数可以作为缓存数据的初始值，返回值可以被dom元素ref标记，可以获取被标记的元素节点.
~~~js
const DemoUseRef = ()=>{
    const dom= useRef(null)
    const handerSubmit = ()=>{
        /*  <div >表单组件</div>  dom 节点 */
        console.log(dom.current)
    }
    return <div>
        {/* ref 标记当前dom节点 */}
        <div ref={dom} >表单组件</div>
        <button onClick={()=>handerSubmit()} >提交</button> 
    </div>
}
~~~
### 高阶用法 缓存数据
- 当然useRef还有一个很重要的作用就是缓存数据，我们知道usestate ,useReducer 是可以保存当前的数据源的，但是如果它们更新数据源的函数执行必定会带来整个组件从新执行到渲染，如果在函数组件内部声明变量，则下一次更新也会重置，如果我们想要悄悄的保存数据，而又不想触发函数的更新，那么useRef是一个很棒的选择。
- useRef 可以第一个参数可以用来初始化保存数据，这些数据可以在 current 属性上获取到 ，当然我们也可以通过对 current 赋值新的数据源
~~~js
// 初始化
const currenRef = useRef(InitialData)
// 获取
const getCurrentData = currenRef.current
// 更改
currenRef.current = newData
~~~

~~~js
import React, { useState, useEffect, useMemo, useRef } from 'react'
export default function App() {
    const dom = useRef([1, 2])
    const [count, setCount] = useState(0)
    useEffect(() => {
        setTimeout(() => {
            // 修改current不会触发函数更新，会在其他会重新渲染函数的hooks搭便车，如useStete
            dom.current =[4, 5]
        },2000)
    },[])
    return(
        <>
            <div>{[...dom.current]}</div>
            <div>{count}</div>
            <button onClick={() => setCount(count +1)} >点击</button>
        </>
        
    )
}
~~~

## useContext 自由获取context
- 我们可以使用useContext ，来获取父级组件传递过来的context值，这个当前值就是最近的父级组件 Provider 设置的value值，**useContext参数一般是由 createContext 方式引入** ,也可以父级上下文context传递 ( 参数为context )。useContext 可以代替 context.Consumer 来获取Provider中保存的value值
~~~js
/* 用useContext方式 */
const DemoContext = ()=> {
    const value:any = useContext(Context)
    /* my name is alien */
return <div> my name is { value.name }</div>
}

/* 用Context.Consumer 方式 */
const DemoContext1 = ()=>{
    return <Context.Consumer>
         {/*  my name is alien  */}
        { (value)=> <div> my name is { value.name }</div> }
    </Context.Consumer>
}

export default ()=>{
    return <div>
        <Context.Provider value={{ name:'alien' , age:18 }} >
            <DemoContext />
            <DemoContext1 />
        </Context.Provider>
    </div>
}
~~~
## useReducer 无状态组件中的redux
- useReducer 是react-hooks提供的能够在无状态组件中运行的类似redux的功能api，至于它到底能不能代替redux react-redux ,我个人的看法是不能的 ，redux 能够复杂的逻辑中展现优势 ，而且 redux的中间件模式思想也是非常优秀了，我们可以通过中间件的方式来增强dispatch redux-thunk redux-sage redux-action redux-promise都是比较不错的中间件，可以把同步reducer编程异步的reducer。useReducer 接受的第一个参数是一个函数，我们可以认为它就是一个reducer ,reducer的参数就是常规reducer里面的state和action,返回改变后的state, useReducer第二个参数为state的初始值 返回一个数组，数组的第一项就是更新之后state的值 ，第二个参数是派发更新的dispatch函数 。**dispatch 的触发会触发组件的更新，这里能够促使组件从新的渲染的一个是useState派发更新函数，另一个就 useReducer中的dispatch**。

~~~JS
const DemoUseReducer = ()=>{
    /* number为更新后的state值,  dispatchNumbner 为当前的派发函数 */
   const [ number , dispatchNumbner ] = useReducer((state,action)=>{
       const { payload , name  } = action
       /* return的值为新的state */
       switch(name){
           case 'add':
               return state + 1
           case 'sub':
               return state - 1 
           case 'reset':
             return payload       
       }
       return state
   },0)
   return <div>
      当前值：{ number }
      { /* 派发更新 */ }
      <button onClick={()=>dispatchNumbner({ name:'add' })} >增加</button>
      <button onClick={()=>dispatchNumbner({ name:'sub' })} >减少</button>
      <button onClick={()=>dispatchNumbner({ name:'reset' ,payload:666 })} >赋值</button>
      { /* 把dispatch 和 state 传递给子组件  */ }
      <MyChildren  dispatch={ dispatchNumbner } State={{ number }} />
   </div>
}

~~~



## useCallback useMemo版本的回调函数
- useMemo和useCallback接收的参数都是一样，都是在其依赖项发生变化后才执行，都是返回缓存的值，区别在于useMemo返回的是函数运行的结果，**useCallback返回的是函数，这个回调函数是经过处理后的也就是说父组件传递一个函数给子组件的时候，由于是无状态组件每一次re-render都会重新生成新的props函数，这样就使得每一次传递给子组件的函数都发生了变化，**这时候就会触发子组件的更新，这些更新是没有必要的，此时我们就可以通过usecallback来处理此函数，然后作为props传递给子组件
~~~js
/* 用react.memo */
const DemoChildren = React.memo((props)=>{
   /* 只有初始化的时候打印了 子组件更新 */
    console.log('子组件更新')
   useEffect(()=>{
       props.getInfo('子组件')
   },[])
   return <div>子组件</div>
})

const DemoUseCallback=({ id })=>{
    const [number, setNumber] = useState(1)
    /* 此时usecallback的第一参数 (sonName)=>{ console.log(sonName) }
     经过处理赋值给 getInfo */
    const getInfo  = useCallback((sonName)=>{
          console.log(sonName)
    },[id])
    return <div>
        {/* 点击按钮触发父组件更新 ，但是子组件没有更新 */}
        <button onClick={ ()=>setNumber(number+1) } >增加</button>
        <DemoChildren getInfo={getInfo} />
    </div>
}
~~~
- 这里应该提醒的是，useCallback ，必须配合 react.memo pureComponent ，否则不但不会提升性能，还有可能降低性能
## useImperativeHandle
- useImperativeHandle 可以配合 forwardRef 自定义暴露给父组件的实例值。这个很有用，我们知道，对于子组件，如果是class类组件，我们可以通过ref获取类组件的实例，但是在子组件是函数组件的情况，如果我们不能直接通过ref的，那么此时useImperativeHandle和 forwardRef配合就能达到效果
- useImperativeHandle接受三个参数：
  第一个参数ref: 接受 forWardRef 传递过来的 ref。
  第二个参数 createHandle ：处理函数，返回值作为暴露给父组件的ref对象。
  第三个参数 deps:依赖项 deps，依赖项更改形成新的ref对象。
- `useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 [`forwardRef`](https://zh-hans.reactjs.org/docs/react-api.html#reactforwardref) 一起使用：
~~~js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
~~~
- 在本例中，渲染 `<FancyInput ref={inputRef} />` 的父组件可以调用 `inputRef.current.focus()`

## useDebugValue

- useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签。这个hooks目的就是检查自定义hooks
~~~js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // ...
  // 在开发者工具中的这个 Hook 旁边显示标签
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
~~~
>
我们不推荐你向每个自定义 Hook 添加 debug 值。当它作为共享库的一部分时才最有价值。在某些情况下，格式化值的显示可能是一项开销很大的操作。除非需要检查 Hook，否则没有必要这么做。因此，useDebugValue 接受一个格式化函数作为可选的第二个参数。该函数只有在 Hook 被检查时才会被调用。它接受 debug 值作为参数，并且会返回一个格式化的显示值。
>
## useTransition
- useTransition允许延时由state改变而带来的视图渲染。避免不必要的渲染。它还允许组件将速度较慢的数据获取更新推迟到随后渲染，以便能够立即渲染更重要的更新。
~~~js
const TIMEOUT_MS = { timeoutMs: 2000 }
const [startTransition, isPending] = useTransition(TIMEOUT_MS)
~~~
- useTransition 接受一个对象， timeoutMs代码需要延时的时间。
- 返回一个数组。第一个参数：  是一个接受回调的函数。我们用它来告诉 React 需要推迟的 state 。 第二个参数： 一个布尔值。表示是否正在等待，过度状态的完成(延时state的更新)。
- 下面我们引入官网的列子，来了解useTransition的使用。
~~~js
const SUSPENSE_CONFIG = { timeoutMs: 2000 };

function App() {
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition(SUSPENSE_CONFIG);
  return (
    <>
      <button
        disabled={isPending}
        onClick={() => {
          startTransition(() => {
            const nextUserId = getNextId(resource.userId);
            setResource(fetchProfileData(nextUserId));
          });
        }}
      >
        Next
      </button>
      {isPending ? " 加载中..." : null}
      <Suspense fallback={<Spinner />}>
        <ProfilePage resource={resource} />
      </Suspense>
    </>
  );
}

~~~
- 在这段代码中，我们使用 startTransition 包装了我们的数据获取。这使我们可以立即开始获取用户资料的数据，同时推迟下一个用户资料页面以及其关联的 Spinner 的渲染 2 秒钟（ timeoutMs 中显示的时间）
- 这个api目前处于实验阶段，没有被完全开放出来。

## 自定义kooks
### 什么自定义Hooks
- 自定义hooks是在react-hooks基础上的一个拓展，可以根据业务需要制定满足业务需要的hooks，更注重的是逻辑单元。通过业务场景不同，我们到底需要react-hooks做什么，怎么样把一段逻辑封装起来，做到复用，这是自定义hooks产生的初衷。
- hooks 专注的就是逻辑复用， 是我们的项目，不仅仅停留在组件复用的层面上。hooks让我们可以将一段通用的逻辑存封起来。将我们需要它的时候，开箱即用即可。
### 自定义hooks-驱动条件
- hooks本质上是一个函数。函数的执行，决定与无状态组件组件自身的执行上下文。每次函数的执行(本质上就是组件的更新)就会执行自定义hooks的执行，由此可见组件本身执行和hooks的执行如出一辙。
- 那么prop的修改,useState,useReducer使用是无状态组件更新条件，那么就是驱动hooks执行的条件。 
- 通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。
- 假设我们现在有两个组件，一个是好友信息组件，根据好友的状态来设置离线、在线。一个是好友列表组件，根据好友状态来设置头像颜色。所以获取好友状态的这一个逻辑就是复用的。
~~~js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
~~~
- 与 React 组件不同的是，自定义 Hook 不需要具有特殊的标识。我们可以自由的决定它的参数是什么，以及它应该返回什么（如果需要的话）。换句话说，它就像一个正常的函数。但是它的名字应该始终以 use 开头，
- 此处 useFriendStatus 的 Hook 目的是订阅某个好友的在线状态。这就是我们需要将 friendID 作为参数，并返回这位好友的在线状态的原因。
### 如何使用
- 我们一开始的目标是在 FriendStatus 和 FriendListItem 组件中去除重复的逻辑，即：这两个组件都想知道好友是否在线。
- 现在我们已经把这个逻辑提取到 useFriendStatus 的自定义 Hook 中，然后就可以使用它了：
~~~js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
~~~
~~~js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
~~~
- 这段代码等价于原来的示例代码吗？等价，它的工作方式完全一样。如果你仔细观察，你会发现我们没有对其行为做任何的改变，我们只是将两个函数之间一些共同的代码提取到单独的函数中。自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。
- 自定义 Hook 必须以 “use” 开头吗？必须如此。这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 Hook 的规则
- 在两个组件中使用相同的 Hook 会共享 state 吗？不会。自定义 Hook 是一种重用状态逻辑的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。
- 自定义 Hook 如何获取独立的 state？每次调用 Hook，它都会获取独立的 state。由于我们直接调用了 useFriendStatus，从 React 的角度来看，我们的组件只是调用了 useState 和 useEffect。 正如我们在之前章节中了解到的一样，我们可以在一个组件中多次调用 useState 和 useEffect，它们是完全独立的。

