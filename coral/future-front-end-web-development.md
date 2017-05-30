# What is the Future of Front End Web Development?

原文地址: [https://css-tricks.com/future-front-end-web-development/](https://css-tricks.com/future-front-end-web-development/)

某一天我被要求对这个话题做个分享。作为单方面的说辞，可以说我还不够资格回答这个问题。如果你真的需要有说服力的答案，你可能要看看各个开发者们怎么说的了。

不过我还是有点点资格说点什么的。除了运行这个网站督促着我每天思考前端开发和需要参加各种前端大会之外，我本身也是一个活跃的开发者。我在前端开发们的聚集地 CodePen 工作。我还每周都会再 ShopTalk Show 和各位来宾分享我的知识，并且我参加的各种开发者大会都基本是前端开发。

好了，我们接着讲今天主题。

再一次声明：

1、我的理解还不够全面
2、这仅仅是我的想法
3、我只是条咸鱼

## User expectations on the rise.

网站现在需要做的事情更多了。开发者们也需求快速的做更复杂的事情了，并且还要又快又好。

## New JavaScript is here.

虽然jquery曾经很棒，但是他确实过时了。虽然es6还没有全面普及到每位开发者，但是这是迟早的事。直接操作dom变得越来越麻烦，现在都习惯用状态管理了。就像我前面提到的，用户期望和程序复杂度都在提升。我们需要让复杂度可控。

`State`是一个大概念。就像我之前说的[When Does a Project Need React?](https://css-tricks.com/project-need-react/)。建站之初就需要考虑哪些状态需要管理，然后为这些状态创建合适的sotre

比如这些框架。Ember, React, Vue, Angular, Svelte等等。他们都是基于状态，组件，并且替我们处理dom了。

现在他们都在速度，特性，和api的简洁性上竞争了。

Typescript看起来是最终的胜者，因为它的兼容性和稳定性，以及给开发者们带来的更好的编辑器体验。

## We're not building pages, we're building systems.

css module，设计模式。这些对于现在的web项目来说慢慢的都快成为标准。系统能构建任何需要的部分。`pages`的概念离我们越来越远了。用户们看到的基本都是基于组件拼接成的。这种拼接工作UX, 交互设计师，甚至销售都能完成。

新的Javascript能做的越来越好了。

## The line between native and web is blurring.

Sketch和Figma哪个更好？我们判断他们是基于他们的特性的，而不管他们是native app还是web app。我是用Slack或者TweetDeck的原生应用，还是打开网页标签？说真的都差不多。有时候web app很棒。我喜欢原生app主要是能在我的桌面上有个图标，然后能保留我的登录状态。

我通常觉得某些功能"需要"原生app才行，但其实在web上也能做到，甚至做得更好。比如音频和视频app，Skype没见缺什么功能，Lightstream在在线播放上也没毛病，Zencaster能记录多音轨并且高质量的声音。这些都是在浏览器上完成的。

这些都是在web上都能做好的典范。web技术正在大跃进。Service workers允许我们离线允许和推送通知。Web Audio API. Web Payments API。web将会是之后app的首要平台。

用户只在乎好不好呀，不考虑app是怎么做出来的。

## URLs are still a killer feature.

web真是做对了这件事。用通用的方法做具体的事是很难的。URL让搜索引擎可行，几乎是人类史上最重要的创新。URL让分享和书签也可行。URL对市场是公平的。任何人都能访问URL，没人管你。

## Performance is a key player.

不再能忍受新能差的网站了。每个人都希望每件事都是立即完成的。

## CSS will get much more modular.

当我们写样式的时候，我们总是要做些抉择。这是全局样式吗？我需要让这个样式是全站可用的吗？还是我要让这些css只对当前组件可用？css将在这两种之间产生最优解。组件专用的样式将会存进组件，并且按需打包的。

## CSS preprocessing will slowly fade away.

许多预处理器已经把它(variables)带到css了，或者更高级的构建处理方式(import)。直到现在我们还是需要工具来模块化和打包css，这些工具已经接管了不少css 预处理器的工作了。现在的预处理器，我认为最重要的是mixins。如果原生css开始实现mixins(有可能是@apply)和继承(有可能是@extend)，css 预处理的作用将会迅速降低。

## Being good at HTML and CSS remains vital.

虽然HTML的构建方式和是否要用dom还没定局。但是你仍然需要知道好的HTML长什么样。知道怎么写良好的html对你是有好处的，对用户和你怎么处理样式都是有好处的。

虽然css在浏览器的处理方式和怎么使用它还在变，但是你仍然需要知道怎么用css。你要知道怎么处理布局，管理空格，调整排版，并且要优雅。

## Build processes will get competitive.

因为性能很麻烦，并且有太多优化方案，我们将看到从代码到生产环境的变革。像webpack这种工具(tree shaking, code splitting)已经做了很多优化，但是还是在到生产环境之前，这些自动化工具还是有很多优化空间。优化第一次加载。处理资源加载顺序。Deciding what gets sent where and how. Shipping nothing whatsoever that isn't used.

随着web平台的进化，构建处理也将会调整，并且会产生最佳事件，我们静观其变。

