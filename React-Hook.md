### 0. 边缘知识补充
1. react16.8出的新特性，在react16.3版本时推出一个新的调和算法 Fiber Reconciler，在16.3之前采用的是Stack Reconciler。Stack(栈)调和主要缺点的是从调和到提交一步到位不能进行中断，假设当调和阶段改动非常大时(diff的组建和patch的布丁非常大)此时动画和交互将会卡顿，这是由于js单线程所致。而Fiber机制可以理解为异步渲染,会把要渲染的任务进行化分优先级、切块、中断、重新执行渲染，每执行一小块会查看是否有空余时间如果有继续执行，如果没有会让出执行权。

2. Fiber如何进行分块和让出执行权呢？
了解这个问题了解渲染一帧需要经历写什么步骤,如下图(图片来源于简书DC_er
)[requestIdleCallback和requestAnimationFrame详解](https://www.jianshu.com/p/2771cb695c81)。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2fb002f015041958bfe4eb0bf348883~tplv-k3u1fbpfcp-zoom-1.image)
浏览器每一帧执行完成可能会有一个空闲时间。例如一帧16ms(1秒60帧约等于16ms)。假设执行完成图上的步骤要10ms，剩余6ms。6ms秒后是另一帧的开始，所以这6ms叫做空闲时间，浏览器可以通过requestIdleCallback知道剩余多少时间，还可以设置一个到期时间，由于可能一直没有空余时间，方法就不会被执行了，所以可以设置一个到期时间，到多久后没有执行则强制执行。具体看链接。这里可以通过这个方法在每次空闲时间执行调和，每一帧都有时间做交互和动画。

3. 下图Fiber的生命周期，具体可以看文档，图片来源于知乎司徒正美[React Fiber架构](https://zhuanlan.zhihu.com/p/37095662)
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3501716a2a9948d08b6e6b77ac5f30bc~tplv-k3u1fbpfcp-zoom-1.image)
其中老得生命期在react17中将会被抛弃，有：componentWillMount、componentWillReceiveProps、componentWillUpdate它们在新的Fiber机制下可能被渲染N次(N>=1)。

4. Fiber Reconciler带来了新的结构Fiber结构，每一个Component都会有一个fiber对象和它对应。这里个人随便打印的一个，里面有个核心熟悉 memoizedState，如果是类对象memoizedState对应的是state对象，如果函数式组件对于的是Hook对象。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9274bae49b404281ae2e6d650b36c4fe~tplv-k3u1fbpfcp-zoom-1.image)

### 1. Hook解决的痛点
1. 从社区的偏好和React的理念来说，函数式组件是被推崇，唯一的输入唯一的输出，单项的数据流，例如Redux的reducer也是，对于大型项目来说便于维护和测试。
2. 可以让纯函数式组件拥有自己的状态、对生命周期的监控(副作用的清理) 、缓冲函数或者数据(类似computed)。
3. 自定义Hook可以更加方便的复用逻辑。例如可以把播放器的逻辑全部抽离出来。(组件复用UI简单、复用逻辑比较难)。解构清晰、不用层层嵌套、使用更加少，更容易阅读。是否想过常见的封装都是UI组件为主，而Hook可以实现状态封装(setState).
4. 解构清晰、不用层层嵌套、使用更加少，更容易阅读。
5. 复杂的组件逻辑。很多的生命周期。hook把生命周期简化了
6. this的指定，但我觉得没问题。
7. 更容易拆分组件。
8. 清除副作用更加紧凑。useEffect

### 2. Hook常用的方法
> Hook大多数方法都有两个参数，一个为***，另一个为空或数组，第二参数的意思就是那些数据改变它需要进行响应。也是优化重点。
1. useState: 为函数式组件提供状态

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2f83c72c9e84515bcd8609181772096~tplv-k3u1fbpfcp-zoom-1.image)

2. useEffects:(异步执行) componentDidMount、componentDidUpdate 的组合体。放回清理函数可选，在卸载时调用。有两个参数，第一个参数可以返回一个函数，第二个参数是一个数组，但数组里的值发生改变时，第一个函数的返回函数回被调用，当为空时，每次都会调用，当为[]是则componentWillUnmount才会调用。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/811255bc29e940c8a628a1cc20a552c4~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9ca31eab2f9434b96cdd5bb55faa9f9~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa93426b5e9949a18cedba195eb2b923~tplv-k3u1fbpfcp-zoom-1.image)

3. [useLayoutEffect](https://www.cnblogs.com/sunidol/p/11301785.html)里面的callback函数会在DOM更新完成后立即执行,但是会在浏览器进行任何绘制之前运行完成,阻塞了浏览器的绘制。
4. useRef: 可以用来获取组件的ref属性还可以当作标记用。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a0e47e2851f41088551ee293e4e156d~tplv-k3u1fbpfcp-zoom-1.image)

5. useMemo: 缓存数据，组件，由于函数组件每次调用，内部的函数和局部变量都会从新生成，当一些数据要经过复杂运算才得出结果时，明显是非常耗性能的。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e4fbfa3428a412b921e867d70e0d190~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e118abe7051e4fbdbeab9336d58f7b25~tplv-k3u1fbpfcp-zoom-1.image)

图一countSqure一直为1，它不会进行重新计算，由于它没有变化，内部的结果被缓存了。这里平时开发要注意。
6 useCallBack：缓存函数，组件，不会每次都重新生成一个函数。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30e27508d71a475db5222092789392ee~tplv-k3u1fbpfcp-zoom-1.image)

和上图一样，count一直不变函数被缓存变量为什么没变呢？(闭包)
7. createContext: 实现父子通信，可以跨级，子可以是第一层子或者N层，原理和类组件getChildContext一样或者类似
```
/**
 * @method createContext
 * 我理解为生成一个全局上下文概念,类似于class组件getChildContext, 
 * 和redux实现类似，redux作者进入了facebook，把redux理念带进去了
 * hook源码充斥着redux信息
 */
const CountContext = createContext()

function A1() {

    // 也可以使用const count = useContext(CountContext) 接受

    return (
        <CountContext.Consumer>
            {
                (count) => (
                    <div>
                        我是A1{count}
                    </div>
                )
            }
        </CountContext.Consumer>
    )
}
function IntroUseReduxPage() {
    const [count, setCount] = useState(1);
    function add() {
        setCount(count+1)
    }
    return (
        <CountContext.Provider value={count}>
            <A1/>
            <button onClick={add}>{count}</button>
        </CountContext.Provider>
    )
}
```

7. useReducer, useContext, useSelector, useDispatch实现一个redux看图就不一一介绍了。其中如果熟悉ant design3.x和4.x人会发现，3.x之前要使用函数包裹组件，而4.x不用，为啥呢，我没看过，但可以实现useReducer实现一套一样的简约版。过几天补上Github实现封装form的链接。
```
import React,{useReducer, useContext, createContext} from 'react'

const Store = createContext()

function AgeComponent() {
    const age = useSelector((state) => state.age);
    const dispatch = useDispatch();
    // 简单点不做优化哈
    const addAge = function() {
        dispatch({type: ACTION_USER_ADD_AGE, data: age+1})
    }
     
    return (
        <div>
            <h2>我的年龄{age}</h2>
            <button onClick={addAge}>加一岁</button>
        </div>
    )
}

function NameComponent() {
    const name = useSelector((state) => state.name);
    const dispatch = useDispatch();
    // 简单点不做优化哈
    const changeName = function(e) {
        dispatch({type: ACTION_USER_CHANGE_NAME, data: e.target.value || 'mochixuan'})
    }
     
    return (
        <div>
            <h2>我的姓名:{name}</h2>
            <input onChange={changeName} placeholder='改个名字吧' />
        </div>
    )
}

//定义两个type
const ACTION_USER_ADD_AGE = 'ACTION_USER_ADD_AGE';
const ACTION_USER_CHANGE_NAME = 'ACTION_USER_CHANGE_NAME';

// reducer 使用react-immutable优化，演示就随便了
function allReducer(state, action) {
    console.warn(action);
    switch(action.type) {
        case ACTION_USER_ADD_AGE:
            return Object.assign({},state, {age: action.data});
        case ACTION_USER_CHANGE_NAME:
            return Object.assign({},state, {name: action.data});
        default:
            return state;
    }
}
// initialState
const initialState = {
    age: 26,
    name: 'mochixuan'
}

// 自己实现redux hook的 useSelector简单版
function useSelector(selector) {
    const {state} = useContext(Store);
    return selector(state);
}
function useDispatch() {
    const {dispatch} = useContext(Store);
    return dispatch;
}

function IntroUseReduxPage() {
    const [state, dispatch] = useReducer(allReducer,initialState);
    const store = {state, dispatch};
    return (
        <Store.Provider value={store}>
            <AgeComponent />
            <br/>
            <NameComponent />
        </Store.Provider>
    )
}



export {IntroUseReduxPage}
```

运行效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b5f17bd83144dc39ebd6648f15d8602~tplv-k3u1fbpfcp-zoom-1.image)

### 3. Hook的缺点
1. 不要在循环，条件判断，函数嵌套中使用hooks
2. 只能在函数组件中使用hooks
3. 很多hook函数都要自己写第二个参数不能自动识别，每次都要自己加关联属性
4. 团队学习成本，几乎改变人们对React的认识

### 4. Hook实现的原理
这里主要会围绕这Hook的存储和数据的更新，涉及到fiber细节暂时不说。这样可以更加清晰的了解hook原理。接下来源码围绕下面这几行代码讲，虽然只有几行代码但完全可以讲解清楚了。将复杂的东西简单化。

```
function IntroSourceCodePage() {
    const [count, setCount] = useState(1);
    
    useEffect(() => {
        console.warn('请调用我 useEffect')
    }, [])
    
    // 把这个写在第三位，证明非hook函数不会影响hook顺序
    const add = function() {
        setCount(count+1);
        setCount(count+2);
        setCount(count+3);
    }
    
    const squareCount = useMemo(()=> count*count, [count])
    
    return(
        <div style={{padding: 40}}>
            <button onClick={add}>{`增加1  count=${count}  squareCount=${squareCount}`}</button>
        </div>
    )
}
```

hook存在哪里呢: 每一个组件都有一个fiber对象(_reactInternalFiber)与其对应，如下图，里面有一个属性memoziedState存储的就是当前的hook对象，hook对象以链表的形式存储的。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9274bae49b404281ae2e6d650b36c4fe~tplv-k3u1fbpfcp-zoom-1.image)

[ReactFiberHooks.js](https://github.com/facebook/react/blob/92fcd46cc79bbf45df4ce86b0678dcef3b91078d/packages/react-reconciler/src/ReactFiberReconciler.new.js) Hook对象包含了什么。

```
export type Hook = {
  memoizedState: any, 是用来记录当前useState应该返回的结果的

  baseState: any,    
  baseUpdate: Update<any, any> | null,  
  queue: UpdateQueue<any, any> | null,  缓存队列，存储多次更新行为,应该是中断形成环 last指向最后一次，next指向第一次

  next: Hook | null, 指向下一次useState对应的Hook对象 
};
```

1. 大致过程，打断点位置

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c317caf5859a467ca03b97b7400862aa~tplv-k3u1fbpfcp-zoom-1.image)

2. 第一次调用useState时hook为空，所以新建一个hook对象挂在fiber的memoziedState上，queue更新队列为空，执行下一行。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f755242b6a904d38ad8274fb92a12ecd~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9488dbfda5ae424cb7742b4826fa1461~tplv-k3u1fbpfcp-zoom-1.image)

执行第二行useEffect

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a97176954234502850458a9bc949af1~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba2b6a12d87645c08a7f75942a4004c3~tplv-k3u1fbpfcp-zoom-1.image)

这里采用循环可能主要是为了统一

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fce9e90bddd4216abb081325566b9c6~tplv-k3u1fbpfcp-zoom-1.image)

执行第三个Hook useMemo，判断数组里的数据和memoziedState[1]是否相同相同则读取memoziedState[0]，不同重新调用函数进行生成。所以当相同是函数是可以不用再调用了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c39ecdde37fa4bd79f116aa39994a5f9~tplv-k3u1fbpfcp-zoom-1.image)

3. 当单击按钮时会向hook的update(更新队列添加一个action可以理解为一个更改，类redux)
代码是C2、C3、C4。会被加入到队列(环形链表) queue-C4-C2-C3-C4-C2。当函数再次执行useState会判断queue是否空，不空则执行，获取C4.next进行更新，当更新到C4时进行退出。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbc38bc649a44b9d90b9c462042d1be3~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10300e9606ca49298103f930de978eed~tplv-k3u1fbpfcp-zoom-1.image)

4. 链表结构

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b64b27768504d03aa1894972af25c57~tplv-k3u1fbpfcp-zoom-1.image)
