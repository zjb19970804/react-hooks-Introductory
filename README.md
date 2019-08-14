# react-hooks介绍
>react-hooks是react在16.8版本中正式推出的API，新API包含有[useState](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate)、[useReducer](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)、[useEffect](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect)、[useContext](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)、[useCallback](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)、[useMemo](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)、[useLayoutEffect](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect)、[useRef](https://zh-hans.reactjs.org/docs/hooks-reference.html#useref)、[useImperativeHandle](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)、[useDebugValue](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedebugvalue)。

- [react-hooks常见的问题](https://zh-hans.reactjs.org/docs/hooks-faq.html)

## 为什么会出现hooks
 - **在组件之间复用状态逻辑很难**<br/>
	`React` 没有提供将可复用性行为“附加”到组件的途径（例如，把组件连接到 `store`）。如果你使用过 `React` 一段时间，你也许会熟悉一些解决此类问题的方案，比如 `render props` 和 高阶组件。但是这类方案需要重新组织你的组件结构，这可能会很麻烦，使你的代码难以理解。如果你在 `React DevTools` 中观察过 `React` 应用，你会发现由 `providers`，`consumers`，`高阶组件`，`render props` 等其他抽象层组成的组件会形成“结构嵌套地狱”。尽管我们可以在 `DevTools` 过滤掉它们，但这说明了一个更深层次的问题：`React` 需要为共享状态逻辑提供更好的原生途径。

- **复杂组件变得难以理解**<br/>
	我们经常维护一些组件，组件起初很简单，但是逐渐会被状态逻辑和副作用充斥。每个生命周期常常包含一些不相关的逻辑。例如，组件常常在 `componentDidMount` 和 `componentDidUpdate` 中获取数据。但是，同一个 `componentDidMount` 中可能也包含很多其它的逻辑，如设置事件监听，而之后需在 `componentWillUnmount` 中清除。相互关联且需要对照修改的代码被进行了拆分，而完全不相关的代码却在同一个方法中组合在一起。如此很容易产生 bug，并且导致逻辑不一致。

	在多数情况下，不可能将组件拆分为更小的粒度，因为状态逻辑无处不在。这也给测试带来了一定挑战。同时，这也是很多人将 React 与状态管理库结合使用的原因之一。但是，这往往会引入了很多抽象概念，需要你在不同的文件之间来回切换，使得复用变得更加困难。

	为了解决这个问题，Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据），而并非强制按照生命周期划分。你还可以使用 reducer 来管理组件的内部状态，使其更加可预测。

- **难以理解的 class**<br/>
	除了代码复用和代码管理会遇到困难外，`class` 是学习 React 的一大屏障。你必须去理解 JavaScript 中 `this` 的工作方式，这与其他语言存在巨大差异。还不能忘记绑定事件处理器。没有稳定的语法提案，这些代码非常冗余。大家可以很好地理解 `props`，`state` 和自顶向下的数据流，但对 class 却一筹莫展。即便在有经验的 React 开发者之间，对于函数组件与 class 组件的差异也存在分歧，甚至还要区分两种组件的使用场景。class 不能很好的压缩，并且会使热重载出现不稳定的情况。
	> 对比下面两段代码
	```
	class ProfilePage extends React.Component {
	  showMessage = () => {
		alert("Followed " + this.props.user);
	  };
	  handleClick = () => {
		setTimeout(this.showMessage, 3000);
	  };
	  render() {
		return <button onClick={this.handleClick}>Follow</button>;
	  }
	}
	```

	>```
	function ProfilePage(props) {
	  const showMessage = () => {
		alert("Followed " + props.user);
	  };
	  const handleClick = () => {
		setTimeout(showMessage, 3000);
	  };
	  return <button onClick={handleClick}>Follow</button>;
	}
	```

	**这两个组件都描述了同一个逻辑：点击按钮 3 秒后 alert 父级传入的用户名。**
	**如果3秒之内，父组件的user值被改变了，上面两个alert分别弹出的是修改前的还是修改后的值？**

<br/>

|  | class | function |
| :- | :-: | :-: |
| 写法 | 复杂，继承自React.Componet,constructor中接受props参数，render中返回react片段 | 简单，直接接受props作为参数，return返回代码片段 |
| state | 可以使用this.state,setState()等 | 无状态组件 |
| 生命周期 | 有 | 无 |
| 优点 | 可以提升性能，有些时候我们需要减少组件的渲染次数，我们就需要在组件内部用shouldComponentUpdate 方法来去判断，或者继承React.PureComponent 类（自动调用shouldComponentUpdate）来实现state和props的浅比较进行判断组件是否重新渲染。 | 无状态组件，更好的体现容器和表现分离，逻辑组件与UI展示组件的解耦 |

## useState

> 代替类组件的state和setState

```
// 简单的初始值
const [count, setCount] = useState(1)
// 复杂的初始值
const [timestamp, setTimestamp] = useState(() => moment(new Date()).add(1, 'd').valueOf())
```

- `useState` 可以接受一个参数来填充初始值， 该参数可以是一个函数，函数必须要有返回值，可以在获取初始值比较复杂的情况下使用
- `useState` 返回两个值：第一个为状态值state，第二个为可以更新所绑定的state的方法

> ** 更新状态的使用示例：**
```
//将count状态设置为2
setCount(2)
```

<br/>

## useReducer

>useState的另一种方案

```
  function reducer(state, action) {
	  switch (action.type) {
		case 'increment':
		  return {count: state.count + 1};
		case 'decrement':
		  return {count: state.count - 1};
		default:
		  return state
	  }
  }

  function init(initialCount) {
  	return {
		count: initialCount
	}
  }

  function Counter({initialCount}) {
  	const [state, dispatch] = useReducer(reducer, initialCount, init);
  }
```
- 接收一个形如 `(state, action) => newState` 的 reducer方法，第二个参数是初始state值。`useReducer` 返回当前的state值与dipatch函数。第三个参数为可选的一个函数，用于初始state较复杂的情况，第二个参数会传入，并返回计算后的值。
- `useReducer` 的适用场景如：state的逻辑较复杂、包含多个子值、依赖之前的state等。由于dispatch函数永远不变，相对于传递一个回调函数给子组件，dispatch的性能优化会好一些。

<br/>

## useEffect

> `useEffect` 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API

```
useEffect(() => {
  const socket = io('http://localhost:3000');
  socket.on('connect', () => //do something)

  return () => {
    socket.close();
  };
}, []);
```

- 上面的`useEffect`接受一个函数和一个数组作为参数， 第一个参数是当effect触发时所执行的callback，第二个参数是effect的依赖项。
- 当其中的任意一个依赖发生改变时就会执行callback，传入的是一个空数组时，只会触发一次该effect，即 `componentDidMount` 。
- 当依赖为空，且return一个函数时，将在该组件销毁时执行return出的函数，即 `componentWillUnmount`

<br/>

## useCallback

> 优化的回调方法

```
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

- 把回调函数及依赖项数组作为参数传入 `useCallback`，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

<br/>

## useMemo

> 优化计算属性

```
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- 把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。
- 记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

<br/>

## useRef

```
const refContainer = useRef(initialValue);
```

- useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

<br/>

## useLayoutEffect

- 其与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染

<br/>

## 实现一个自定义可重用逻辑的Hook

```
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

然后在其他组件中引入：
```
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```
> 这两个组件的 state 是完全独立的。Hook 是一种复用状态逻辑的方式，它不复用 state 本身。事实上 Hook 的每次调用都有一个完全独立的 state —— 因此你可以在单个组件中多次调用同一个自定义 Hook。

<br/>


### 在类组件中使用可上拉加载长列表的示例
```
class List extends React.Component {
  didCancel = false
  
  state = {
	pageNum: 0,
	pageSize: 10,
	total: 0,
	hasMore: true,
	loading: true,
	data: []
}
  componentDidMount() {
    this.loadList()
  }

  componentWillReceiveProps(nextProps) {
    do_something(nextProps);
  }

  componentWillUnmount() {
    this.didCancel = true
  }

  async loadList() {
	const { loading, pageNum, pageSize, data } = this.state
	if(loading) return
	this.setState({ loading: true })
    const res = await fetch(`http://xxxxxxxxx?pageNum?pageSize*****`)
	!this.didCancel && this.setState({
		total: res.total,
		hasMore: res.totalPage > nextPage,
		loading: false,
		data: data.concat(res.data)
	})
  }
  
  pullUpHandle() {
  	this.setState((prevState) => ({ pageNum: prevState.pageNum + 1 }), () => this.loadList())
	//做一些其他什么事。。。
  }
  
  changeSize(value) {
  	this.setState({
		pageSize: value
	}, () => this.loadList())
	//做一些其他什么事。。。
  }

  render() {
    return (
		<div>
			<SomeComponent data={data} pullUpHandle={this.pullUpHandle} changeSize={this.changeSize} />
		</div>
	);
  }
}
```

### 现在我们使用react-hooks重写试试
```
const reducer = (state, action) => {
	const { type, payload } = action
	switch(type) {
		case 'GET_LIST':
			const { total, hasMore, data }
			return {
				...state,
				total,
				hasMore,
				data: state.data.concat(data)
			}
		case 'CHANGE_NUM':
			return {
				...state,
				pageNum: payload
			}
		case 'CHANGE_SIZE':
			return {
				...state,
				pageSize: payload
			}
		default：
			return state
	}
}
const initState = {
	pageNum: 0,
	pageSize: 10,
	total: 0,
	hasMore: true,
	data: []
}
function Component() {
  let didCancel = false;
  const [loading, setLoading] = useState(true)
  const [pagState, dispatch] = useReducer(reducer, initState)

  useEffect(() => {
  	return () => {
		didCancel = true
	}
  }, [])

  useEffect(() => {
	if(loading) return
	setLoading(true)
	const res = await fetch(`http://xxxxxxxxx?pageNum?pageSize*****`)
	setLoading(false)
	const payload = { total: res.total, hasMore: res.totalPage > pageNum, data: res.data }
	!didCancel && dispatch({ type: 'GET_LIST', payload })
  }, [pageNum, pageSize]);

  const pullUpHandle = useCallBack(() => {
  	dispatch({ type: 'CHANGE_NUM', payload: pageNum + 1 })
  }, [pageNum])

  const changeSize = useCallBack((value) => {
  	dispatch({ type: 'CHANGE_Size', payload: value })
  })
    return (
		<div>
			<SomeComponent data={data} pullUpHandle={this.pullUpHandle} changeSize={this.changeSize} />
		</div>
	);
}
```

> **你会发现业务逻辑都被抽离到组件外的reducer中了，组件内部只需处理基础的事件触发**

<br/>

#### 最后我使用过程中碰到的一些坑：
- 当在异步请求中使用多个state更新函数时，react不会批量更新。解决方法是使用useReducer或使用[use-immer](https://github.com/immerjs/use-immer)库。
- 不要在useState中直接使用props中的值，除非你确保该值永远不会被更新，因为useState只会在初始时设置initState值。解决方法是在Effect中更新state，依赖为props中你需要的那个值。
> 示例：
```
const [num, setNum] = useState(0)
useEffect(() => {
	setNum(props.value)
}, [props.value])
```
- 最后一点，不要在循环或者条件语句中使用hooks，只需记住将hooks放在组件的最外层，并且保证每次出现调用的顺序一致。如果你不想执行该Effect，可以在Effect里面判断，然后return。

