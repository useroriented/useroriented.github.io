# 一个更好的react架构

现在的react我相信都在用redux或者flux架构，你也能很容易的在github上找到很多已存在的项目的目录结构，或者你也可能会直接使用别人写好的boilerplate。
就拿redux架构举例，你的目录结构可能是这样的:

```js
actions/
api/
components/
containers/
middlewares/
reducers/
store/
routes.js
```

难道你不觉得这种目录很啰嗦很累吗，当你需要加一个新功能的时候：

+ 新建一个action
+ 新建一个reducer
+ 新建一个component
+ 可能还需要新建一个中间件

当你需要修改某个功能的时候，你又需要重新在每个目录中找到那些文件，然后一个个改掉需要修改的部分。这还不包括store配置的时间。

所以我建议大家使用原生的react直接写项目，或者可以借助接近于原生体验的数据流管理库[meng](https://github.com/huangbinjie/meng)。假如你接受了我的建议，那么我们继续整理目录。

我在之前一篇文章介绍了视图、布局和组件的概念[think-of-react-data-flow.](https://github.com/useroriented/useroriented.github.io/blob/master/coral/think-of-react-data-flow.md)，所以我理想中的目录是这样的：

```js
api/
components/
  item/
    item.tsx
    ite.style.ts
views/
  dashboard/
    dashboard.tsx
    dashboard.style.tsx
layouts/
app.tsx
app.style.ts
```

项目地址在：[https://github.com/huangbinjie/react-meng-boilerplate](https://github.com/huangbinjie/react-meng-boilerplate)
这里的app.tsx就是程序入口了。app.tsx里面的路由所对应的组件就是视图，这里是dashboard。目录结构基本上就是参考的[duck modules](https://github.com/erikras/ducks-modular-redux)。
下面是dashboard.tsx的代码:

```js
import * as React from 'react'
import {lift, resource} from 'meng'
import Banner from '../../components/banner/banner'
import Navbar from '../../components/navbar/navbar'
import Item from '../../components/item/item'
import {get} from '../../api/goods'

import {CONTAINER} from '../../app.style'
import {UL} from './dashboard.style'

@resource(get(), "lists")
@lift({ lists: [] })
export default class Dashboard extends React.Component<any, any> {
  render() {
    const lis = this.props.lists.map((list, index) => <Item data={list} key={index}/>)
    return (
      <div>
        <Banner />
        <Navbar />
        <ul className={`${CONTAINER} ${UL}`}>
          {lis}
        </ul>
      </div>
    )
  }
}

```

从代码里我们可以分析视图的职责：获取数据并把数据传给组件和排版。因为我这里没有特别的布局需求，暂时就把布局写在视图里了(ul那段).
样式方面没有选择传统的[css modules](https://css-tricks.com/css-modules-part-1-need/)，而是使用了[free-style](https://github.com/blakeembrey/free-style),
这么做的好处就是没必要依赖webpack。

再来看看组件item.tsx的代码:

```js
import * as React from 'react'
import {CONTAINER, RIGHT} from '../../app.style'
import {LI, HEADER, BADGE, TITLE, DELIVER_PRICE, IMG, LABEL, P, VOTE, GO} from './item.style'


export default class Item extends React.Component<any, any> {
  render() {
    const data = this.props.data
    const tags = data.tags.map((tag, index) => <a key={index}>{tag}</a>)
    return (
      <li className={LI}>
        <header className={HEADER}>
          <a className={BADGE} href="#">{data.fromWhere}</a>
          <a className={TITLE}>{data.title}<span className={DELIVER_PRICE}>{data.totalFee}包邮</span></a>
        </header>
        <a href="#"><img className={IMG} src={data.picUrl} alt=""/></a>
        <div className={LABEL}>标签: &nbsp; &nbsp; {tags}<span className={RIGHT}>{data.time}</span></div>
        <p className={P}>{data.brief}</p>
        <footer>
          <button className={VOTE}>买！{data.mai}</button>
          <button className={VOTE}>不买！0</button>
          <a className={GO} href={data.buyLink}>直达链接</a>
        </footer>
      </li>
    )
  }
}

```

作为组件，它的职责就很简单了，就是展示数据和交互数据，需要展示的数据是从视图传下来的。我知道有很多人喜欢直接让组件加载数据，也不能说不好，至少某些场景有点尴尬。
使用这种react架构之后我开始集中精力在写组件和api上，经历过redux之后难得得舒爽！

以上就是我在用的目录结构。相对于redux的功能型的结构，这种结构相对于更容易理解和上手。如果你也对redux有点疲倦了，希望你能有所收获:-)