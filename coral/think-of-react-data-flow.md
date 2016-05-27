## 简介
体验过flux结构的各大库，比如flux，reflux，redux之后，实在受不了他们的繁琐了，文件夹和文件多得不要不要的。是时候实现下自己的思路了。

### 组件
按照组件化思维，everything is component。前端只有组件。但是组件也是分很多种类型的。

#### 视图
视图是一种高阶组件，视图有以下职责：
##### 组织布局
假如有个需求，需要把一种排版切换到另一种排版，而css完不成这个任务。这里我们可以理解为有两个视图，每个视图有不同布局，随后我们只需要切换视图=>加载当前视图对应的布局
一般情况下，我们没有这个应用场景，这个时候所谓布局就是视图本身了。

##### 加载组件
这里的意思是往布局组件里面填充组件

##### 传递数据
我们知道组件获取数据而不借助其他库的时候有[两种方式](http://andrewhfarmer.com/react-ajax-best-practices/)，分别是组件自己获取和从上层组件传下来。这里我强烈建议从视图获取数据，然后传给通过props传给每个需要这些数据的组件。和视图相关的都应该交给视图做

#### 布局
布局也是一种高阶组件，布局本身不带数据，理论上只带UI状态，只是辅助组件定位，布局不是必须的

#### 一般组件
这就是我们dumb组件了

### 状态
按照react给render的定义，组件只需要props和state就能展现dom结构，props和state统称为状态，这些状态又可以细分为下面三种状态。

#### 组件状态
组件状态指的是组件本身用到的一些变量，比如组件的名字等等

#### UI状态
UI状态指所有的样式

#### 数据状态
渲染视图需要用到的数据

-----------------------------------------------------------------分割线-----------------------------------------------------------------

以上是我对react的理解，我按照我的理解设计和实现了我的库，下面介绍下我的库的设计思想和使用方法.

上面提到组件只需要各种状态就能呈现视图。所以在我的架构基本是就是组件和数据源之间的关系了。一个简单的例子：
app.js
```

import { ajax } from 'rxjs/observable/dom/ajax'
import { connect, resource } from 'meng'

@resource(ajax.get("/user"), "userInfo")
@connect(state => ({ userInfo: state.userInfo }))
class View extends React.Component {
  render() {
    return <div><User data={userInfo}/></div>
  }
}
```
user.js
```
class User extends React.Component {
  render() {
    const {userInfo} = this.props
    return <div>
      nick: {userInfo.nick}
    </div>
  }
}
```
我的库里共有三个方法：
+ connect connect方法为目标组件生成一个状态容器，可以通过第一个参数把state中的变量注入到组件的props里面
+ resource 这个方法可以给组件注入数据源，可以是primitive value也可以是Observable。如果是普通值会直接注入到props，如果是Observable，会订阅Observable然后注入到props里面。
+ getStore 这个方法会返回Store。默认导出的和这个方法一样

### 一些概念

#### state
一个组件对应一个store，这个store也就是这个组件的状态，所有的状态都应该存在这里。store里暂时只有一个方法setState，这个方法是默认注入到组件的props里面的。当调用setState之后，组件是会立即刷新的。当组件生成的时候会在Store里面生成自己的store，组件unmounted之后，这个store也会被删除。其实就是组件和页面的对应关系，不过我的结构是扁平的，不像react是递归的。

#### props
和其他库不一样(比如redux)，我这里区分props和state。props特指父组件传下来的属性，state指的就是组件对应的store，和react的state概念一样，不过我把这个state拿出来放在其他地方了。

#### 组件通讯
组件通讯也有2种方式，一种走父组件(cyclejs)，一种走事件中心(flux)。但是我的做法不一样，我们可以直接修改一个组件的状态:
```
Store.App.setState({})
```

### 和redux对比
我们知道redux是一种多对一的关系，多个组件对应一个store。
和redux的多对一不一样，我这里是多对多，但是又可以交叉。没有redux的action和reducer和初始化，可以直接修改另一个组件的状态，但是也没办法实现redux的middleware了，看取舍咯。

### 和cyclejs对比
作为终极架构cyclejs，我实在没法挑剔的，但是唯一不舒服的是，cyclejs的组件通信需要走父组件回调机制，虽然cyclejs有办法避免一层层传的尴尬，但是还是没办法无视掉啊...
而我的库组件通讯是完爆cyclejs的，至于其他的...被cyclejs完爆.../(ㄒoㄒ)/~~

所以准确的说我的目标是做成rxjs和react的关系库2333

