# 我为什么不用redux

随着redux的架构发展，大家或许都从心底里认为，react就是要ui和logic分离。redux的好处各位想必比我清楚，但是redux的缺点是什么，我需要无视这些缺点而继续使用redux吗？先不管我说得对不对，请继续往下看。

## redux 的缺点

不用redux那就要拿redux开刀，为什么不用redux，因为它的这些缺点：

1、 action和reducer太繁琐。一套或者几套action和reducer的组合，看起来很不错，但是一旦功能和需求多了，action和reducer就会很混乱，如果管理不善，都不能愉悦的写代码了。比如你一个人开发你自己的博客的时候，独自面对大量的action和reducer和store，不知道会不会头疼。所以redux是不适合小场景的。微服务好，也不是什么系统都适合的。

2、store和state的模棱两可。没有严格的定义哪些存store，哪些存internal state。如果不是资深redux玩家，想必也说不出个所以然来。

3、dispatch是同步的，而且dispatch没办法确认action是否执行成功

首先要承认的是redux是非常棒的框架，但是只适合资深redux玩家和中大型项目。

## 状态管理

那么没有了redux，我们应该怎么管理状态。我觉得应该保留redux的精华:store,去掉action和reducer。直接把react直接当做reducer，理想中的循环是这样的:

```js
store --> state --> react(store) --> nextState -> store
```

用装饰器可以得到更直观从上到下的store:

```js
const store: (initialState: object) => React.Component

@store({todos: [], display: "active"})
class App extends React.Component
```

还应该有一个setState方法，而且是异步的:

```js
store.setState(object, callpack)
```

store并不是总是要我们手动setState状态，还应该自动获取effect, 之后根据设置的key自动修改store的状态

```js
@effect(() => fetchTodos(), "todos")
@store({todos: []})
```

连react都有`ComponentWillReceiveProps`，我们当然也需要一个`willReceiveStoreState`:

```js
@willReceiveStoreState((currentState, nextState) => nextState.pid !== currentState.pid ? fetchList(pid), "list")
@store({pid: number, list: []})
```

## 使用store构建应用

上面我们定义了一个store，现在我们需要用这个store，来构建应用了。

![todo app](https://raw.githubusercontent.com/useroriented/useroriented.github.io/master/images/thinking-in-mengbi/todo.png)

用todo来举例，我们对todo视图按功能划分为：header、list、control。再加上视图Todo，一共是4个组件，2种类型。

### 视图

```js
export default class Todo extends React.Component {
  public render() {
    return (
      <section className="todoapp">
        {Header(this.props.todos)}
        <List todos={this.props.todos} />
        <Control display={this.props.display} todos={this.props.todos} />
      </section>
    )
  }
}
```

视图的职责是获取数据，布局，和分配数据。所以`Todo`最终是这样的:

```js
//todos: 任务列表， display: 显示哪种状态的任务
@store({todos: [], display: "active"})
export default class Todo extends React.Component {
  public render() {
    return (
      <section className="todoapp">
        {Header(this.props.todos)}
        <List todos={this.props.todos} />
        <Control display={this.props.display} todos={this.props.todos} />
      </section>
    )
  }
}
```

我们还有个从localstorage读取缓存的api

```js
const getByCache = () => new Promise((resolve, reject) => resolve(JSON.parse(localStorage.getItem("todos"))))
```

把api注入到store里面去

```js
@effect(() => getByCache(), (currentState, cache) => cache)
@store({todos: [], display: "active"})
export default class Todo extends React.Component {
  public render() {
    return (
      <section className="todoapp">
        {Header(this.props.todos)}
        <List todos={this.props.todos} />
        <Control display={this.props.display} todos={this.props.todos} />
      </section>
    )
  }
}
```

### 业务组件

从视图拿到数据之后，我们的业务组件就需要制订业务和分配业务了:

#### header

```js
export const Header = (todos: TTodo[]) =>
  <header className="header">
    <h1>todos</h1>
    <input className="new-todo" onKeyDown={onkeydown(todos)} placeholder="What needs to be done?" autoFocus={true} />
  </header>

const onkeydown = todos => event => {
  const value = event.currentTarget.value
  if (value === "") return
  if (event.keyCode === 13) {
    const newtodos = [...todos]
    newtodos.push({ status: "active", value })
    event.currentTarget.value = ""
    Store.children.Todo.setState({ todos: newtodos }, () => localStorage.setItem("meng-todo", JSON.stringify(this.props)))
  }
}
```

header组件主要处理输入，处理完之后把新的状态交给store就行了。

#### list

```js
export class List extends React.Component {
  public render() {
    const items = this.props.todos.map((todo, index) => (
      <li className={todo.status} key={index}>
        <div className="view">
          <input className="toggle" type="checkbox" onChange={this.toggle(index)} checked={todo.status === "completed" ? true : false} />
          <label>{todo.value}</label>
          <button className="destroy" onClick={this.destroy(index)}></button>
        </div>
      </li>
    ))
    return (
      <section className="main">
        <input className="toggle-all" type="checkbox" />
        <label htmlFor="toggle-all">Mark all as complete</label>
        <ul className="todo-list">
          {items}
        </ul>
      </section>
    )
  }
```

list组件主要负责显示。header输入新的状态，并交给store之后，这里也会响应式的刷新。

在redux里面，组件只是页面的一块方形区域，仅仅只是视图。在去掉redux之后，这里的组件就是业务的集合，包括render和handler。对视图的划分也是按照功能来划分的。

上面的例子我介绍了使用store构建的应用的数据处理流程

+ 视图获取数据
+ 业务组建拿到需要的部分数据处理加工然后合并到store
+ 业务组件从视图拿到新的数据显示或处理

## 使用rxjs实现store

因为`Observable`天然流的特性方便我们抽象和合并的各种各样的数据源。懒执行设计，非常适合store这种push模式，也就是数据驱动视图。我决定使用rxjs来实现store

### Store

rxjs的`BehavorSuject`刚好有Observable的可订阅又有Subject的next(setState)。可以说是完美适合写store了。下面是一段伪代码：

```js
import {BehavorSubject} from "rxjs"

const store = initialState => component => {
  return class extends React.Component {
    hasStoreStateChanged = false
    componentDidMount() {
      const currentStore = new BehavorSubject(initialState)
      currentStore
      .merge(effects$)
      .subscribe(state => {
        this.hasStoreStateChanged = true
        this.setState(state)
      })
    }
    componentShouldUpdate() {
      return this.hasStoreStateChanged
    }
    render() {
      this.hasStoreStateChanged = false
      return React.createElement(component, this.state)
    }
  }
}
```

### effect

effect函数是把所有类型的数据源都抽象成Observable的函数，方便我们的store合并, 下面是effect的伪代码:

```js
const effect = obj => toObservable(obj)

function toObservable(source: any) {
  if (source == void 0)
    return Observable.never()

  else if (source instanceof Observable)
    return source

  else if (source instanceof Promise)
    return Observable.fromPromise(source)

  else if (source instanceof ImplStore)
    return source.store$
  else
    return Observable.of(source)
}
```

至此，一个响应式的store就这样完成了。这也是我利用rxjs实现store的主要思路.

以上所有代码都可以在这里找到: [todo](https://github.com/huangbinjie/meng/tree/master/example/todo)

## 最后

不要试图说服我redux是最棒的。因为我和你的观点是一样的。redux是现阶段最好的架构！

用rxjs配合react的方法有很多种，有继承: `extends StoreComponent`。有组合: `store(ReactComponent)`。也有直接修改react的: `onClick(): click$`。所以我这种实现方式还有很多其他版本的。

最近发现一款国外的库，思路挺相似的，推荐看看[freactal](https://github.com/FormidableLabs/freactal)。