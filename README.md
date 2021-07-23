### 前言

在hooks没有出现之前，函数组件基本只做展示功能，涉及到状态管理，我们需要用到class组件或者数据管理框架（redux/dva/mobx）;

hooks组件趋向函数化编程。更简单，组件颗粒度更小，代码也更少。class组件有如下不足：

| 问题                           | 解决     | 缺点       |
| ------------------------------ | -------- | ---------- |
| 生命周期复杂，逻辑耦合         | 无       |            |
| 大型组件逻辑难以复用，难以拆分 | 继承解决 | 不能多继承 |
|                                |          |            |

而 hooks 对这些问题都有较好的解决方案

* 通过自定义hooks解决多继承
* 通过自定义 useEffect 来解决复用问题，细分逻辑，减小出现逻辑臃肿的场景

为了解决这种，类组件功能齐全却很重，纯函数也有重大限制。也就有了React Hooks，也就是加强版的函数组件，我们可以完全不使用 `class`，就能写出一个全功能的组件。

### hooks常用API讲解

#### useState

`hooks` 的能力，就是让我们在函数组件中使用 `state`, 就是通过 `useState` 来实现的。

`useState` 会返回一个数组，第一个值是我们的 `state，` 第二个值是一个函数，用来修改该 `state` 的。

请看下边Demo：

```
import React, { useState } from "react";
import "./styles.css";

export default function App() {
  const [count, setCount] = useState(0);

  const addOne = () => {
    setCount(count + 1);
  };

  const addTwo = () => {
    setCount((pre) => pre + 2);
    console.log(count);
  };

  return (
    <div>
      点击次数: {count}
      <br />
      <button onClick={addOne}>点我+1</button>
      <br />
      <button onClick={addTwo}>点我+2</button>
    </div>
  );
}
```

`useState(defaultValue)` `defaultValue`可以一个值，也可以是一个函数，返回一个值，，如下`setState((previewState => { return newState }))`也可以这样来更新状态（异步操作）



#### useEffect

`Effect Hook` 可以让你在函数组件中执行副作用操作，什么是副作用呢，就是除了状态相关的逻辑，比如网络请求，监听事件，查找 `dom`

`useEffect`支持两个参数，第一个参数是函数

第二个参数，有三种情况

* 什么都不传，组件每次 render 之后 useEffect 都会调用，相当于 componentDidMount 和 componentDidUpdate
* 传入一个空数组 [], 只会调用一次，相当于 componentDidMount 和 componentWillUnmount
* 传入一个数组，其中包括变量，只有这些变量变动时，useEffect 才会执行
  

`useEffect` 实现生命周期

`useEffect `可以看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个声明周期函数的组合。

    import React, { useState, useEffect } from "react";
    import "./styles.css";
    
    function Child() {
      const [count, setCount] = useState(0);
      const [width, setWidth] = useState(document.body.clientWidth);
    
      const onChange = () => {
        setWidth(document.body.clientWidth);
      };
    
      useEffect(() => {
        // 相当于 componentDidMount
        console.log("componentDidMount");
        window.addEventListener("resize", onChange, false);
    
        return () => {
          // 相当于 componentWillUnmount
          console.log("componentWillUnmount");
          window.removeEventListener("resize", onChange, false);
        };
      }, []);
    
      useEffect(() => {
        // 相当于 componentDidUpdate
        console.log("componentDidUpdate");
        document.title = count;
      });
    
      useEffect(() => {
        console.log(`count change: count is ${count}`);
      }, [count]);
    
      return (
        <div>
          页面名称: {count}
          <br />
          页面宽度: {width}
          <br />
          <button
            onClick={() => {
              setCount(count + 1);
            }}
          >
            点我
          </button>
        </div>
      );
    }
    
    function App() {
      const [visible, setVisible] = useState(true);
    
      return (
        <div>
          <button
            onClick={() => {
              setVisible(!visible);
            }}
          >
            点我{visible ? "隐藏" : "显示"}
          </button>
          {visible && <Child />}
        </div>
      );
    }
    export default App;


#### useContext

`context` 实例中的 `Provider` 和 `Consumer`，在类组件和函数组件中都能使用，`contextType` 只能在类组件中使用，因为它是类的静态属性，具体如何使用 `useContext` 呢，看下面的例子

```
import React, { useState, useContext, Component, createContext } from "react";
import "./styles.css";

// 创建一个 context
const Context = createContext(0);

// 组件一, Consumer 写法
class Item1 extends Component {
  render() {
    return <Context.Consumer>{(count) => <div>{count}</div>}</Context.Consumer>;
  }
}

// 组件二, contextType 写法
class Item2 extends Component {
  static contextType = Context;
  render() {
    const count = this.context;
    return <div>{count}</div>;
  }
}
// 组件一, useContext 写法
function Item3() {
  const count = useContext(Context);
  return <div>{count}</div>;
}

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      点击次数: {count}
      <button
        onClick={() => {
          setCount(count + 1);
        }}
      >
        点我
      </button>
      <Context.Provider value={count}>
        <Item1></Item1>
        <Item2></Item2>
        <Item3></Item3>
      </Context.Provider>
    </div>
  );
}

export default App;
```

三种写法都能够实现我们的需求，但是，三种写有各自的优缺点，下面为对比出的结果

| 写法        | 优缺                                                         |
| ----------- | ------------------------------------------------------------ |
| consumer    | 嵌套复杂，Consumer 第一个子节点必须为一个函数，无形增加了工作量 |
| contextType | 只支持 类组件，无法在多 context 的情况下使用                 |
| useContext  | 不需要嵌套，多 context 写法简单                              |

结论：useContext写法简单，功能丰富



#### useMemo

useMomo一版是用来做优化使用的，可以和Vue的计算属性做对比

`useMemo` 也支持传入第二个参数，用法和 `useEffect` 类似

1. 不传数组，每次更新都会重新计算
2. 空数组，只会计算一次
3. 依赖对应的值，当对应的值发生变化时，才会重新计算(可以依赖另外一个 `useMemo` 返回的值)



#### useCallBack

useCallback 是 useMemo 的语法糖。在 react 中我们经常面临一个子组件渲染优化的问题， 尤其是在向子组件传递函数props时，props包含函数时，每次 render 都会创建新函数，导致子组件不必要的渲染，浪费性能。这个时候，就需要使用是 useCallback ，useCallback 可以保证，当依赖值没有变化是，无论 render 多少次，我们的函数都是同一个函数，减小不断创建的开销。

```
import { useMemo, useState, useCallback, memo } from "react";
import "./styles.css";

const Child = memo(({ value }) => {
  console.log("child 渲染", value);
  return <div>{value}</div>;
});

const Child1 = memo(({ value, add }) => {
  console.log("child1 渲染", value);
  return (
    <div>
      <button onClick={add}>递增m</button>
      {value}
    </div>
  );
});

export default function App() {
  const [n, setN] = useState(0);
  const [m, setM] = useState(0);

  const add = useCallback(() => {
    setM(m + 1);
  }, [m]);

  const n1 = useMemo(() => {
    return n + 1;
  }, [n]);

  return (
    <div className="App">
      <button
        onClick={() => {
          setN((n) => n + 1);
        }}
      >
        递增n
      </button>
      <Child value={n}></Child>
      <div>n+1={n1}</div>
      {/* <button onClick={add}>新增m</button> */}
      <Child1 value={m} add={add}></Child1>
    </div>
  );
}
```

参数同`useMemo`

#### useRef

有两点使用

1. 获取组件的实例（class组件）
2. 在函数组件中的一个全局变量，不会因为重复 `render` 重复申明， 类似于类组件的 `this.xxx`，（也可以理解UI不依赖的数据）

```
import React, {
  useMemo,
  useState,
  useRef,
  PureComponent,
  forwardRef
} from "react";
import "./styles.css";

const Children1 = forwardRef((props) => {
  const { count } = props;
  return <div>{count}</div>;
});

class Children extends PureComponent {
  render() {
    const { count } = this.props;
    return <div>{count}</div>;
  }
}

function App() {
  const [count, setCount] = useState(0);
  const childrenRef = useRef(null);
  const children1Ref = useRef(null);
  const inputRef = useRef(null);

  const onClick = useMemo(() => {
    return () => {
      console.log("button click");
      console.log(childrenRef.current.props);
      console.log(children1Ref.current);
      setCount((count) => count + 1);
    };
  }, []);

  return (
    <div>
      点击次数: {count}
      <Children ref={childrenRef} count={count}></Children>
      <Children1 ref={children1Ref} count={count} />
      <button onClick={onClick}>点我</button>
      <br />
      <input
        ref={inputRef}
        onChange={() => {
          console.log(inputRef.current.value);
        }}
      />
    </div>
  );
}

export default App;
```

#### `useImperativeHandle`

useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值，也就是子组件可以选择性的暴露给父组件一些方法，这样可以隐藏一些私有方法和属性，官方建议，useImperativeHandle应当与 forwardRef 一起使用，具体如何使用看下面例子

```
import React, {
  useCallback,
  useState,
  useRef,
  useImperativeHandle,
  forwardRef
} from "react";
import "./styles.css";

const Child = forwardRef((props, ref) => {
  const introduce = useCallback(() => {
    console.log("子组件 introduce方法");
  }, []);

  useImperativeHandle(ref, () => ({
    introduce: () => {
      introduce();
    }
  }));

  return <div>子组件{props.count}</div>;
});

function App() {
  const [count, setCount] = useState(0);
  const childRef = useRef(null);
  const child1Ref = useRef(null);

  const onClick = useCallback(() => {
    setCount((count) => count + 1);
    console.log(childRef.current);
    console.log(child1Ref.current);
    childRef.current.introduce();
  }, []);

  return (
    <div>
      点击次数: {count}
      <Child ref={childRef} count={count}></Child>
      <button onClick={onClick}>点我</button>
    </div>
  );
}

export default App;
```

#### 自定义hook

自定义hooks可以将组件逻辑复用，颗粒度更小。也是用来解决class组件逻辑难以复用的问题

具体使用请看以下Demo：

```
import React, { useEffect, useState, useCallback } from "react";
import "./styles.css";

function useMousePosition() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });

  const onMouseMove = useCallback((event) => {
    const { clientX, clientY } = event;
    setPosition({
      x: clientX,
      y: clientY
    });
  }, []);

  useEffect(() => {
    window.addEventListener("mousemove", onMouseMove, false);
    return () => {
      window.removeEventListener("mousemove", onMouseMove, false);
    };
  }, []);

  return position;
}

function useDomSize() {
  const [size, setSize] = useState({
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight
  });

  const onResize = useCallback(() => {
    setSize({
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight
    });
  }, []);

  useEffect(() => {
    window.addEventListener("resize", onResize, false);
    return () => {
      window.removeEventListener("resize", onResize, false);
    };
  }, []);

  return size;
}

export default function App() {
  const size = useDomSize();
  const position = useMousePosition();
  return (
    <div className="App">
      <h1>
        宽：{size.width}, 高：{size.height}
      </h1>
      <h2>
        x：{position.x}, y：{position.y}
      </h2>
    </div>
  );
}
```

### jsx

* jsx是js语法的扩展，jsx可以生成React元素。
* jsx中可以html，css，js表达式
* jsx其实是一个对象，和React.createElement生成的对象一模一样

>[jsx官方文档]([https://react.docschina.org/docs/introducing-jsx.html](https://react.docschina.org/docs/introducing-jsx.html)
)

### 元素渲染
```
const element = <h1>Hello, world</h1>;
```
上边是一个元素，如果需要渲染到页面上需要通过`react-dom`插件
```
ReactDom.render(ReactElement，Element)
```
* ReactElement是react对象
* Element是Dom对象

> [官方文档]([https://react.docschina.org/docs/rendering-elements.html](https://react.docschina.org/docs/rendering-elements.html)
)

### 组件（函数/类组件）&组件生命周期&组件间通信

组件的使用首字母必须是大写

**函数组件**

定义组件最简单的方式函数组件：
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

**类组件**
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
两个组件页面展示相同
函数组件单一制作渲染，类组件不仅仅只做渲染，最重要的是有生命周期和组件更新


### 状态&props

**属性**

函数组件接收属性（可以是任意的js变量）
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
类组件接收属性
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

**状态**

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 0};
  }

  handleClick = () => {
    console.log('this is:', this);
    const { count } = this.state;
    this.setState(count: ++count);
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>{this.state.count}</h2>
        <button onClick={handleClick}>add</button>
      </div>
    );
  }
}
```

**生命周期**

* componentWillMount 在渲染前调用,在客户端也在服务端。

* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。

* componentWillReceiveProps 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。

* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
可以在你确认不需要更新组件时使用。

* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。

* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。

* componentWillUnmount在组件从 DOM 中移除之前立刻被调用

![生命周期图解](https://pic3.zhimg.com/v2-09f698c70d89d72b146653ce67f79c0c_1440w.jpg?source=172ae18b)

### 事件处理

* React 事件绑定属性的命名采用驼峰式写法，而不是小写。
* 如果采用 JSX 的语法你需要传入一个函数作为事件处理函数，而不是一个字符串(DOM 元素的写法)

HTML 通常写法是：
```
<button onclick="activateLasers()">
  激活按钮
</button>

```
React 中写法为：
```
<button onClick={activateLasers}>
  激活按钮
</button>
```

**Demo**
```
class Popper extends React.Component{
    constructor(){
        super();
        this.state = {name:'Hello world!'};
    }
    
    preventPop(name, e){    //事件对象e要放在最后
        e.preventDefault();
        alert(name);
    }
    
    const clcik = () => {
    
    }
    
    render(){
        return (
            <div>
                <p>hello</p>
                {/* 通过 bind() 方法传递参数。 */}
                <a onClick={this.preventPop.bind(this,this.state.name)}>Click</a>
                <a onClick={this.clcik}>Click</a>
            </div>
        );
    }
}
```
### 条件渲染，列表，组件通信
**条件渲染**
变量
```
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}

class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}
```

&&
```
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

三元表达式
```
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn
        ? <LogoutButton onClick={this.handleLogoutClick} />
        : <LoginButton onClick={this.handleLoginClick} />
      }
    </div>
  );
}
```

**列表**
```
function ListItem(props) {
  // 正确！这里不需要指定 key：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在数组的上下文中被指定
    <ListItem key={number.toString()}              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
**组件通信**

### 实现to do list DEMO
```
function ToDoItem ({data, deteleToDo, markDone}) {
  return (
    <li>
      <span style={data.done ? {textDecoration: 'line-through', color: '#ccc'} : {}}>{data.name}</span>
      <a style={{marginLeft: 15}} onClick={deteleToDo}>删除</a>
      {
        !data.done && <button style={{marginLeft: 15}} onClick={markDone}>已完成</button>
      }
      
    </li>
  )
}
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      list: [],
      value: ''
    };
  }
  
  addToDo = () => {
    const { list, value } = this.state;
    const data = {
      name: value
    };
    list.push(data);
    this.setState({list: [...list]}, () => {
      this.setState({value: ''})
    })
  }
  
  deteleToDo = (index) => {
    const { list } = this.state;
    list.splice(index, 1);
    this.setState({list: [...list]});
  }
  
  markDone = (index) => {
    const { list } = this.state;
    list[index].done = true;
    this.setState({list: [...list]});
  }
  
  inputOnChange = (event) => {
    const value = event.target.value;
    this.setState({value});
  }
  
  render() {
    const { list, value } = this.state;
    return (
      <div>
        <h1>TO DO LIST</h1>
        <div style={{display: 'flex'}}>
           <input value={value} onChange={this.inputOnChange} />
           <button onClick={this.addToDo}>添加</button>
        </div>
        <ul>
        {
          list.map((item, index) => {
            return (
               <ToDoItem data={item} 
                         key={index} 
                         deteleToDo={() => {
                          this.deteleToDo(index);
                         }} 
                         markDone={() => {
                          this.markDone(index);
                         }}
               />
            )
          })
        }
        </ul>
      </div>
    );
  }
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);


```
