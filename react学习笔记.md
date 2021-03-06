# React学习笔记

React是一个声明式，高效灵活的用于构建用户界面的JavaScript库。

举例，创建一个包含2个子节点的DOM节点：

```javascript
// 原生JavaScript
let oDiv = document.createElement('div'),
    oP = document.createElement('p'),
    oSpan = document.createElement('span');
oDiv.className = 'box';
oP.innerText = 'This is a p';
oSpan.innerText = 'This is a span';
oDiv.appendChild(oP);
oDiv.appendChild(oSpan);

// React JS
const ele = React.createElement(
    'div',
    {className: 'box'},
    React.createElement('p', null, 'This is a p'),
    React.createElement('span', null, 'This is a span')
);

// JSX语法(JavaScript和XML)
const ele = (
    <div className='box'>
    <p>this is a p</p>
    <span>this is a span</span>
    </div>
);
```

`React.createElement(type[, props][, ...children])`是一个函数，参数`type`是节点类型（如`div`、`p`等），props为节点属性，需要传一个JS对象，其余参数为子节点（传字符串即为文本节点，传React元素对象即为元素节点），返回一个React元素节点。

得到React元素节点后，可以借助如下函数进行渲染：`ReactDOM.render(element, container[, callback])`在提供的container中渲染一个React元素，并返回对该组件的引用（无状态组件返回null）。如果提供了可选的回调函数，则会在组件被渲染后执行。

从上面例子中可以看出JSX的清晰简洁性，但JSX语法浏览器不能识别，所以通常写JSX语法后需要编译称为浏览器能识别的React JS语法。

编译JSX可以利用`<script src="..."></script>`来引入`babel-standalone`，在写JSX的语法的`script`标签中添加`type="text/babel"`属性，这样浏览器就能识别其中的JSX语法并将其转换为React JS语法。例如：

```html
<script src="https://rivalsa.github.io/Subminiature/code/react-16_13_1/babel_min.js"></script>
<script type="text/babel">
	// ...
</script>
```

要想使用React还需要再引入react和react-dom两个库，前者主要负责UI界面，后者主要负责DOM操作。例如：

```html
<script src="https://rivalsa.github.io/Subminiature/code/react-16_13_1/react_development.js"></script>
<script src="https://rivalsa.github.io/Subminiature/code/react-16_13_1/react-dom_development.js"></script>
```

如果想将完整的库下载到本地，可以利用node.js下载，如下命令供参考：

```bash
yarn add babel-standalone react react-dom
```

## JSX语法

JSX会被自动转换为React元素对象。

JSX中指定类名时应写作`className`，指定表单的`value`时应写作`defaultValue`，`label`标签的`for`属性应写作`htmlFor`

JSX在表单中使用设置`value`属性时，此属性值不会随用户动作（比如在`type="text"`框中输入数据）而变化，它只会随代码中设置的属性值而变化。所以通常通过监听表单的`onChange`事件来实时变更`value`的属性值，例如：

```jsx
class App extends React.Component {
  state = {
    currentValue: ''
  }
  handleChange(e) {
    this.setState({
      currentValue: e.target.value
    });
  }
  render() {
    return (
      <>
        <input type="text" value={this.state.currentValue} onChange={this.handleChange.bind(this)} />
        {this.state.currentValue && <h1>{this.state.currentValue}</h1>}
      </>
    );
  }
} 
```

JSX指定行内样式时需要用大括号包裹一个样式对象，例如：

```jsx
const ele1 = (
    <div style={{color: "red", backgroundColor: "skyblue"}}>一段文字</div>
);
// 或者
const myStyle = {
    color: 'red',
    backgroundColor: 'skyblue'
};
const ele2 = (
    <div style={myStyle}>一段文字</div>
);
```

插值符号`{}`可以将某个变量值或表达式值直接放入对应位置，对于数组，则会依次插入所有元素，无需遍历，例如：

```jsx
const a = '我是一段文字',
    b = ['数组元素1', '数组元素2'],
    ele = (
        <div>
            <p>{a}</p>
            <p>{b}</p>
        </div>
    );
```

JSX中的内容要符合XML规则，所有标签都必须是双标签，例如：

```jsx
{/* 正确 */}
<br />

{/* 正确 */}
<br></br>

{/* 错误 */}
<br>
```

JSX中只能有一个根节点，如果需要绑定多个空节点，可以用`<React.Fragment>...</React.Frement>`套起来（`<>...</>`是它的语法糖），例如：

```jsx
{/* 正确 */}
<React.Fragment>
    <div className="box">
        <h1>hello JSX</h1>
    </div>
    <p>this is a p</p>
</React.Fragment>

{/* 正确 */}
<>
    <div className="box">
        <h1>hello JSX</h1>
    </div>
    <p>this is a p</p>
</>

{/* 错误 */}
<div className="box">
    <h1>hello JSX</h1>
</div>
<p>this is a p</p>
```

在遍历数组时，需要给每个元素绑定一个独一无二的key，例如：

```jsx
const names = ['aaa', 'bbb' , 'ccc'],
      ele = (
          <div>
              {
                  names.map((item, i) => {
                      return (
                          <p key={i}>{i}:{item}</p>
                      );
                  })
              }
          </div>
      );
```

绑定事件时只能绑定行内事件，事件名称用小驼峰表示，事件函数用插值符号引入即可，例如：

```jsx
const ele = (
    <h1 onClick={fn}>test</h1>
);
```

## 组件

使用React可以将一些简短、独立的代码片段组合成复杂的UI界面，这些代码片段称为**组件**。

在JSX中可以通过`<组件名 属性=值 属性=值...>`的方式来调用组件（其中`属性=值 ...`部分可以全部替换为`...`运算符），组件名的首字母必须大写。

### 函数组件

一个函数（函数名首字母大写），返回一段JSX就可以理解为一个函数组件，例如：

```jsx
function Fn(props) {
    console.log(props);
    return (
        <div>
            <h1>{props.dream}</h1>
        </div>
    );
}
// 可以通过<Fn dream="test"/>来调用这个组件
```

### 类组件

通常创建一个继承`React.Component`的类作为组件，类名就是组件名（必须首字母大写）

在类中需要创建一个名为`render`的公共函数，此函数的返回值为该组件实际要渲染的内容（此函数为组件生命周期中的一环，必有）。

类中可以有一个`state`对象，表示组件的状态，此对象可以用`setState`方法来修改其中的内容，在修改后会自动重新渲染修改后的内容。

`setState(updater[, callback])`将对组件 state 的更改排入队列，并通知 React 需要使用更新后的 state 重新渲染此组件及其子组件。`updater`可以是一个对象，也可以是一个返回对象的函数，`setState`执行后会根据这个对象，或函数返回的对象来修改`state`，并重新渲染。如果`updater`是一个函数，那么执行时会传2个实参，第一个是`state`，第二个是调用组件时传的参数。`callback`是回调函数。

```jsx
class Fn extends React.Component {
    state = {
        dream: '我是dream'
    }
    myClick() {
        this.setState({
            dream: '新dream'
        });
    }
    render() {
        return (
            <div>
                <h1 onClick={this.myClick.bind(this)}>{this.state.dream}</h1>
            </div>
        );
    }
}
```

可以通过`this.props`来获取调用组件时设置的属性及属性值。如果组件标签内部还有其他元素，也会通过此属性获取对应元素。

 在jsx中可以通过`ref`属性来绑定通过`React.createRef()`创建的ref，此时可以通过ref值操作这个元素对象吗，例如：

```jsx
export default class App extends React.Component {
  myRef = React.createRef()
  handleClick() {
    this.myRef.current.style.backgroundColor = 'hotpink';
  }
  render() {
    return (
      <h1 ref={this.myRef} onClick={this.handleClick.bind(this)}>test</h1>
    );
  }
}
```

### 组件之间的通信

父组件向子组件通信通常在父组件调用子组件时，利用属性来进行传递信息，在子组件中利用`props`来进行接收。

子组件是不能直接向父组件通信的，但可以通过如下方案来变通的实现这一功能：在父组件中定义一个函数将这个函数通过属性传递给子组件，在子组件中通过调用函数传参来使父组件获得传递的参数值。**这个方法本质上还是父组件向子组件通信，但借助函数实现了子组件的数据传递给父组件。**

同级组件之间也不能直接传递，但可以借助其共同的父组件进行传递，数据先有子组件传递至父组件，再由父组件传递至另一个子组件。

### 组件的生命周期

**创建组件时**

1. constructor
2. getDerivedStateFromProps（静态方法，我没搞清楚怎么用）
3. render（渲染函数，唯一不能省略的函数，必须有返回值，返回null或false表示不渲染任何DOM元素）
4. React更新DOM和refs
5. componentDidMount（挂载成功函数）

**更新props或执行setState时**

1. getDerivedStateFromProps（静态方法，我没搞清楚怎么用）
2. shouldComponentUpdate（返回一个布尔值，true表示继续执行生命周期，false表示不再执行生命周期，有2个参数，分别为`nextProps`和`nextState`）
3. render（渲染函数）
4. getSnapshotBeforeUpdate（此函数必须有返回值，返回值将最为第6环的第三个参数，有2个参数，分别为`prevProps`和`prevState`）
5. React更新DOM和refs
6. componentDidUpdate（更新完成函数，有3个参数，分别为`prevProps`，`prevState`和`snapshot`）

**卸载组件时**

componentWillUnmount

## Hooks

Hooks可以让无状态的组件（函数组件）获得一些状态，它包括如下函数，运行后会返回对应的对象（来自React）：

- useHistory
- useLocation
- useParams
- useRouteMatch
- useState会返回一个数组，第一个元素是当前状态值，第二个元素为一个函数，执行此函数可以更新状态。执行此`useState`函数需要传一个参数为初始的状态值。
- useEffect执行此函数时需要传一个函数类型的参数，当状态发生更新后会自动调用这个函数参数；还可以传第二个参数（可选）为数组，仅当数组中对应的元素的内容发生了变化，才会执行第一个函数参数，如果传一个空数组则表示尽在组件挂载和卸载时执行。

## 路由包

当利用React创建单页面网站时可以借助路由包让页面随URL的不同而不同。可以参考如下命令通过node.js下载路由包。

```bash
yarn add react-router-dom
```

使用路由前应通过`import`引入包，当使用`import`引入时，应使用解构的方式调用相关内容，例如：

```jsx
import {...} from 'react-router-dom'
```

也可以不下载路由包，而直接在HTML通过`script`标签引入，例如：

```html
<script src="https://rivalsa.github.io/Subminiature/code/react-16_13_1/react-router-dom_min.js"></script>
```

**`BrowserRouter`组件与`HashRouter`组件**

路由有Browser路由和Hash路由两种，采用前者页面中需要被路由部分应包含在标签`<BrowserRouter></BrowserRouter>`中，采用后者页面中需要被路由部分应包含在标签`<HashRouter></HashRouter>`中。

**以下介绍的组件必须包含这两个标签的其中一个之内。**

**`Redirect`组件**

通过此组件可以让页面在前端重定向，此组件需要提供`to`属性，属性值为要重定向到的页面，与`from`属性，属性值为从那个路由开始重定向。

**`Link`组件**

此组件需要指定`to`属性，属性值可以为字符串，也可以为对象。为字符串时要填写转跳到的路由位置（URL地址），如`<Link to="/user">用户中心</Link>`；为对象时，可以包含`pathname`属性、`search`属性、`state`属性。

此组件可以设置`exact`属性，设置了此属性表示匹配时采取全字匹配，没有此属性则表示可以部分匹配。

**`NavLink`组件**

此组件的基本功能、用法与`Link`组件相同。不一样之处在于当页面路由与此组件`to`属性值相同时，页面会为此元素增加以下两个属性：`aria-current="page"`、`class="active"`。

**`Route`组件**

此组件需要指定`path`属性，属性值为路由位置（可以使用`*`表示匹配所有路由），当当前页面的路由位置与此属性值匹配时，会显示此标签内的内容，例如：

```jsx
<Route path="/user">
    <h1>这里是个人中心</h1>
</Route>
```

当前页面的路由位置与此属性值匹配时需要显示的页面也是一个组件时，除了类似上例的方法外，还可以通过如下2种方法：

```jsx
{/* 方法1 */}
<Route Path="..." component={组件或函数} />
{/* 方法2 */}
<Route Path="..." render={函数} />
{/* 函数返回的组件为要渲染的内容，执行函数时会传一个实参props */}
```

此组件的路径中还可以通过`:字段`的方式来获取URL中对应位置的信息。当通过`Route`的`compontent`属性或`render`属性调用其他组件时，此信息将以对象的形式传递到调用的组件的`props`信息中。

> `prop`信息是一个对象，其中包含`location`属性、`history`属性、`match`属性、`StaticComtext`属性。
>
> `history`中包括页面前进、后退以及相关的函数。
>
> `location`属性值为对象，里面记录这页面的URL信息，如：`hash`、`pathname`、`search`、`state`、`query`。
>
> `match`中记录的是`:字段`在URL中对应的信息。

此组件可以设置`exact`属性，设置了此属性表示匹配时采取全字匹配，没有此属性则表示可以部分匹配。

**`Switch`组件**

当`Route`组件包含在`Switch`标签内时，遇到第一个能匹配的路由后就停止匹配，如没有包含在此组件内，则无论是否匹配成功都会一直匹配到最后。

## Redux

### Redux包

Redux包是用来做状态管理的，它所提供的状态可以在多个组件中使用，避免了组件间通信的问题。可以参考如下命令通过node.js下载Redux包。

```bash
yarn add redux
```

从此组件中可以解构出`createStore`函数，执行此函数可以创建一个状态，此函数需要传递一个函数参数，返回一个对象。

> **传递的函数参数**
>
> 传递的函数参数中通常规定了当执行某个“指令”时如何修改状态，此函数有两个形参，第一个为`state`，是一个对象，表示当前状态，第二个为`action`，也是一个对象，表示执行的“指令”，例如：
>
> ```javascript
> const store = createStore((state = {count: 0}, action) => {
>   switch(action.type) {
>     case 'jia':
>       return Object.assign({}, state, {count: state.count + 1});
>     case 'jian':
>       return Object.assign({}, state, {count: state.count - 1}); 
>     default:
>       return state;
>   }
> });
> ```
>
> **整个函数的返回值**
>
> 返回值为一个对象，包括如下内容：
>
> - `getState`函数，执行此函数可以获得当前的状态对象。
> - `dispatch`函数，执行此函数需要传一个“指令”对象，系统会将当前状态与此“指令”对象作为实参，执行创建状态时传递的函数参数，并将返回值作为当前的状态。
> - `subscribe`函数，此函数需要传一个函数参数，当此函数被执行时，会对状态的变化进行监听，当状态发生变化时，会自动执行参数中传递的函数。

**修改此状态后通常页面不会被重新渲染，需要靠组件的`setState`来让其重新渲染**

### React-Redux包

 React-Redux是结合了React的Redux，使用React-Redux需要安装Redux与React-Redux两个包，可以参考如下命令通过node.js下载Redux包。

```bash
yarn add react-redux
```

此组件中可以解构出`Provider`组件，需要用此组件包含需要使用状态的组件，此组件需要提供属性`store`，其属性值为`createStore`函数的返回值。

此组件中还可以解构出`connect`函数，通过此函数可以将状态对应到组件的`props`中。执行此函数需要传两个参数（用到哪个传哪个就行），第一个参数为一个函数，参数为当前的状态，返回值会加入到`props`中，第二个参数为一个函数，参数为修改状态的函数，返回值为一个由函数组成的对象，这些函数会加入到props中，如果不传第二个参数，则会在`props`中携带一个`dispatch`函数，用于修改状态。整个函数的返回值为一个函数，执行此函数需要传一个参数（组件），返回值为含有对应`props`内容的组件。

## React脚手架

可以参考如下命令来安装脚手架

```bash
yarn global add create-react-app
```

可以参考如下命令来初始化脚手架项目

```bash
create-react-app 项目名
```

> 如果以上命令出现错误，可以将对应命令的目录加入环境变量`path`中，可参考如下命令查看全局安装目录
>
> ```bash
> yarn global dir
> ```

可以参考如下命令来运行脚手架项目

```bash
yarn start
```

可以参考如下命令来编译脚手架项目

```bash
yarn build
```
