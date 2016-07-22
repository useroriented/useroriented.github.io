# Design Principles

当你在很多项目里用过react之后，你可能想对react贡献代码。在[行动](https://github.com/facebook/react/blob/master/CONTRIBUTING.md)之前，我们认为决定公开一些react的设计原则。

我们写这篇文章是为了告诉大家react做了的和没做的和我们的开发理念是什么样的。我们很高兴看到大家贡献的代码，但是我们不可能选择那些和我们设计原则背道而驰的建议。

> ## 注意
> 这篇文章假设你是一名很懂react的开发者，它描述的是react的设计原则，而不是讲组件或者程序的
> 如果是新手，请先看[Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)

## 构成

React最关键的特性是组件的自由组合。不同的人写的组件组合在一起也能正常工作。给一个组件增加功能不会让整个代码库都要修改。

比如你可以改变一个组件本地状态而不必要改变使用这个组件的组件。还有你也可以随时增加和删除一些组件的代码。

在组件中使用state和生命周期函数没什么不好的。像任何强大的特性一样，他们也应该被适度的使用，但是我们不打算移除他们。相反，我们认为正是他们让react变得更有用。
我们未来也许会开放更多的[功能](https://github.com/reactjs/react-future/tree/master/07%20-%20Returning%20State)，但是本地状态和生命周期会一直存在。

组件通常被大家理解成"函数",但是在我们的观点中，组件可比函数有用多了。在React中，组件用来描述任何可组合的行为，包括渲染，生命周期，本地状态。一些外部库，比如Relay给组件增加了其他功能--数据依赖。

## Common Abstraction

我们一般很抵制一些用户能自己实现的[特性](https://www.youtube.com/watch?v=4anAwXYqLG8)。我们不想给你们的app增加额外而无用的代码而让你们的app变得臃肿。但是，还是有些例外。

比如，如果React不提供本地状态和生命周期函数，用户一般会自己去实现它们。当多个功能互相冲突的时候，react就不能从性能中获益。

这就是为什么我们有时候又会给React增加一些特性。如果我们发现许多组件需要一个特性，而你们实现起来有冲突或者低效率的话，我们就有可能把这项特性带到React中。
我们不会草率的下决定。如果我们决定要做，那么我们就能确性提升抽象等级对整个生态圈是有益处的。状态，生命周期，跨浏览器事件兼容都是这么来的。

我们会在社区中和大家一起讨论改进建议。你能在github issue搜索[big picture](https://github.com/facebook/react/issues?q=is%3Aopen+is%3Aissue+label%3A%22big+picture%22)找到这些帖子

## Escape Hatches

React是实用主义的。是被Facebook的产品需求驱动的。React被一些范式影响着，比如不太主流的函数式编程，让不同技能和经验的程序员们都能理解它是react的目标。

如果我们打算摒弃一些我们不喜欢的模式的时候，我们有责任考虑所有可能用到的情况和弃用之前[告诉社区可替代的方案](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html)
如果一些有用的模式很难通过声明式的方式呈现给大家，我们会提供[命令式版本的api](https://facebook.github.io/react/docs/more-about-refs.html)。
如果我们提供一个正式的api，那么我们就会提供一个[临时可用的api](https://facebook.github.io/react/docs/context.html)直到正式api出来之前。

## Stability

我们会评估API的稳定性。在FB，我们有超过2万个组件正在用react。许多其他用户，包括Twitter和Airbnb都是React的重度用户。这就是为什么我们不太情愿修改公共API和行为的原因。

However we think stability in the sense of "nothing changes" is overrated. It quickly turns into stagnation. Instead, we prefer the stability in the sense of "It is heavily used in production, and when something changes, there is a clear (and preferably automated) migration path."

当我们反对一个模式的时候，我们会研究他在FB的使用率和增加一些警告。这样会帮助我们评估改变造成的影响。如果为时尚早那我们就需要想出更多的方式应让我们的代码库能适应这个改变

如果我们发现某项改变还不算糟糕，而且后期迁移修改也很方便，我们就会在开源社区公布这个反对警告。我们和FB之外的React用户保持在紧密的联系，对于那些流行的开源项目，我们会帮助他们修复将出的修改

鉴于FB代码库庞大的体积，成功的内部改动差不多可以说明其他公司也没问题。但是总有一些情况是我们没考虑到的，所以我们会增加escape hatches或者重新思考我们之前的方式是否合理

我们不会无缘无故的摒弃任何事情。我们承认这些警告有时候会带来挫败感，但是我们是在为开发体验和新特性铺路，其中很多都是社区里认为值得加进去的。

比如我们在15.2.0的时候加的[未知DOM属性警告](https://facebook.github.io/react/warnings/unknown-prop.html).很多项目都受到影响了。然后修复这个警告可以让我们为React引进[自定义属性](https://github.com/facebook/react/issues/140)的功能。这就是个个警告背后的故事之一。

当我们增加一条反对警告的时候，我们会给当前主要版本留点时间，然后在[下个主要版本修改](https://facebook.github.io/react/blog/2016/02/19/new-versioning-scheme.html)。如果涉及到大量且重复的手工内容，我们会放出[codemod](https://www.youtube.com/watch?v=d0pOgY8__JM)脚本自动化这些操作。
Codemods让我们能往前走而不用停下了，我们也建议你们也这么做。

你能在[react-codemod](https://github.com/reactjs/react-codemod)仓库里找到我们发布的codemods

## Interoperability

对于已存在的系统中的互通性我们也是高度重视的。FB有大量的非react代码库。站点使用的是混合的服务端组件系统叫做XHP，内部UI也是非react代码和react代码混合的。让一个团队为[一个小特性使用react](https://www.youtube.com/watch?v=BF58ZJ1ZQxY)而不是重写他们的代码对我们来说是很重要的目标。


## Scheduling

虽然你的组件只是一个函数，当你使用React的时候你也没必要直接调用这个函数。每个组件都会返回[需要渲染的内容](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html#elements-describe-the-tree),这些内容可能包括用户写的组件`<LikeButton>`和默认组件`<div>`.
React会在之后的递归中根据结果自动调用`<LinkButton>`然后改变UI树

这是一个很小的特性，但是非常有用。你不需要调用组件函数，交给React去做，也就意味着React有能力延迟调用它直到React觉得需要调用了。现在版本的React会调用整个更新数的render方法，然后递归生成新树。
未来我们可能会[延迟更新避免丢帧](https://github.com/facebook/react/issues/6170)

这是React设计里基本的主题。在新数据生成的时候，一些主流库都是实现"push"方式完成计算。然后React是"pull"模式，知道需要计算的时候React才会计算。

React不仅仅是一个数据处理库，它是为构建用户视图而生的。这是React独特的地方--让app知道哪些需要计算哪些不需要。

我们能延迟一些相关逻辑。如果速度获取速度快于画面刷新率，我们会合并然后一起处理他们。我们会优先处理用户操作(比如点击按钮产生的动画),再处理优先级较低的底部工作(比如渲染从接口获取到的数据)以避免丢帧。

但是我们现在还没有完全这样做。我们希望能hold住计划，这就是为什么setState()是异步的。我们认为这是"打算更新"而不是立即更新

如果我们让用户用fp里常见的"push"模式直接组合视图，那控制住这些计划对我们来说会越来越难。

让用户的代码做到最小化执行是一个很关键的目标。React保留了计划的能力，分隔工作任务到个个块中。

在我们团队中有个笑话，React应该叫做"Schedule"，因为React不是完全的响应式的。
