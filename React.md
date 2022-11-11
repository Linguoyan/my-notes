# React 版本特性



## React18



### 1.自动批处理



#### **什么是批处理？**



React 将多个状态更新合并到单次「重新渲染」中，防止多个状态触发多次重新渲染，以便获得更好的性能。

~~~js
// 如同一个点击事件中有两个状态更新，React 总是将它们分批处理到一个重新渲染中
const [count, setCount] = useState(0);
const [flag, setFlag] = useState(false);

function handleClick() {
    setCount((c) => c + 1); // 还没有重新渲染
    setFlag((f) => !f); // 还没有重新渲染
    // React 只会在最后触发一次重新渲染（这是批处理！）
}

return (
    <div>
        <button onClick={handleClick}> Next </button>
        <h1 style={{ color: flag ? "blue" : "black" }}> {count} </h1>
	</div>
);
~~~

这十分有利于性能的提高，因为避免了不必要的渲染。就好像点菜时，服务员不会在你点完一道菜就跑到厨房告诉厨师，而是等你点完菜单。



但有个问题，React 只在浏览器事件期间批量更新，在异步（promise/settimeout）回调的事件中，React 不会对状态更新进行批处理。

~~~js
function handleClick ( )  { 
    asyncEvent().then(res => {
      // React 17 及更早版本不会对这些进行批处理
      setCount (c  =>  c  +  1);  // 导致重新渲染
      setFlag (f  =>  ! f); // 导致重新渲染
    })
  }
~~~



#### **自动批处理**



React 推出 `createRoot` 定义渲染节点，使用该 API 后，所有更新都将自动批处理，无论更新来自何处。以便用户在程序中获得更好的性能。



#### **退出自动批处理**



用 `ReactDOM.flushSync()` 选择退出批处理。

~~~js
flushSync(() => {
    setCounter(c => c + 1);
});
~~~



### 2.startTransition



#### 什么是过渡？



慢速渲染以达到 UI 过渡的效果。比如搜索关键词联想的场景，通常用户键入一个词就得获取对应的联想词数据，如果用户连续输入了好几个词，那么多次联想词的获取和渲染就变得没有必要，我们可以在用户连续输入好几个字后，再获取联想数据。



我们将状态更新分为两类：

- 紧急更新：更新后直接交互，如打字、悬停、拖动等。
- 过渡更新：将 UI 从一个视图过渡到另一个视图，如关键词联想。



#### 解决了什么问题？



对于关键词搜索过滤列表数据的场景：

~~~js
// 紧急：显示输入的内容 用户每输入一个词就立即更新
setInputValue(input) ;

// 不急：显示结果 可以等待用户完成输入后再更新
setSearchQuery(data) ;
~~~



**在 React18 之前，所有状态更新都是紧急渲染。**这意味着上面例子中的输入值和结果数据都会同时渲染。但过滤结果获取并不是紧急的，我们可以通过 `startTransition` 来实现慢速渲染：

~~~js
import { startTransition } from 'react';

// 紧急：立即显示输入的内容
setInputValue(input) ;

// 慢速渲染：非紧急处理，如果出现更紧急的更新，则会中断
startTransition (() => { 
  // Transition: 显示结果
  setSearchQuery(data); 
});
~~~

包装在 startTransition 里的更新被视为非紧急处理，如果出现紧急更新（比如用户又输入了值），那么上面的更新将会被中断。

如果用户停止转换，React 渲染最新状态。

Transitions 通过 UI 的过渡实现慢速渲染，避免不必要的更新和性能浪费。



# 性能优化



可将性能优化分为两个大的分类：

- 加载时优化
- 运行时优化



## 加载时优化



顾名思义就是让一个网站加载速度更快，耗时更短。比如压缩文件大小、使用 CDN 加速等方式优化加载性能。

检查加载性能的指标一般看：白屏时间和首屏时间。



白屏时间计算

~~~js
// 将代码脚本放在 </head>
<script>
    new Date().getTime() - performance.timing.navigationStart
</script>
~~~



首屏时间计算

~~~js
// 在 window.onload 事件中执行以下代码，可以获取首屏时间
new Date().getTime() - performance.timing.navigationStart
~~~



### 优化方式



从 URL 输入到页面渲染完成过程：

1. 交给 DNS 域名解析，找到对应的 IP 地址
2. 进行 TCP 连接
3. 浏览器发送 HTTP 请求
4. 服务器接收请求，处理请求，并返回 HTTP 响应报文
5. 浏览器接收并解析渲染页面



从该过程可以发现可优化的点：

1. DNS 解析优化，浏览器访问 DNS 的时间得到缩短
2. 使用 HTTP2
3. 减少 HTTP 请求数量
4. 减少 HTTP 请求大小
5. 服务器端渲染
6. 静态资源使用 CDN
7. 资源缓存，不重复加载相同的资源



#### 1.DNS 预解析



`DNS Prefetching ` 是具有此属性的域名不需要用户点击链接就在后台解析，而域名解析和内容载入是串行的网络操作，所以这个方式能减少用户的等待时间，提升用户体验。



具体实现

~~~js
// 用meta信息来告知浏览器, 当前页面要做DNS预解析
<meta http-equiv="x-dns-prefetch-control" content="on" />
    
// 在页面header中使用link标签来强制对DNS预解析
<link rel="dns-prefetch" href="http://bdimg.share.baidu.com" />
~~~



#### 2.使用 HTTP2



#### 3.减少请求数量



如果页面的资源非常碎片化，每个 HTTP 请求只带回来几 K 甚至不到 1K 的数据（比如各种小图标）那性能是非常浪费的。



#### 4.压缩、合并文件



- 压缩文件：减少HTTP请求大小，可以减少请求时间
- 文件合并：减少HTTP请求数量



**文件压缩**



我们可以对 html、css、js 以及图片资源进行压缩处理，用 webpack 实现：

- js 压缩：uglify-plugin、terser-webpack-plugin
- CSS 压缩：optimize-css-assets-plugin
- HTML 压缩：html-webpack-plugin
- 图片压缩：image-webpack-loader



**文件合并**



提取公共代码。什么样的文件可以合并呢？可以提取项目中多次使用到的公共代码进行提取，打包成公共模块：

~~~js
optimization: {
    splitChunks: {
        chunks: 'all',
        name: true,
    },
},
~~~



#### 5.采用 svg 图片或字体图标



字体图标或者 SVG 是矢量图，是通过代码编写出来的，放大不会失真，而且渲染速度快，生成的文件特别小。



#### 6.按需加载、减少冗余



**按需加载**



根据文件内容生成文件名，结合 import 动态引入组件实现按需加载：

~~~js
output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].js',
    path: path.resolve(__dirname, '../dist'),
},
~~~



**减少冗余**



`babel-loader` 用  `include` 或 `exclude` 来帮我们避免不必要的转译。开启 `cacheDirectory` 将公共文件缓存起来，加快二次编译。



#### 7.服务器端渲染



客户端渲染：获取 HTML 文件，根据需要下载 JavaScript 文件，运行文件，生成 DOM，再渲染。

服务端渲染：服务端返回 HTML 文件，客户端只需解析 HTML。

优缺点：首屏渲染快，SEO 好。缺点：配置麻烦，增加了服务器的计算压力。



#### 8.使用 Defer 加载JS



所有放在 head 标签里的 CSS 和 JS 文件都会堵塞渲染。如果这些 CSS 和 JS 需要加载和解析很久的话，那么页面就空白了。

因此 CSS 文件放头部，避免用户第一时间看到没有样式的「丑陋」页面；JS 文件放底部，等 HTML 解析完了再加载 JS 文件。

JS 文件也可以放在头部，但要给 script 标签加上 defer 属性，异步下载，延迟执行。



#### 9.静态资源使用 CDN



#### 10.图片资源优化



雪碧图：图标合并；



图片懒加载：先将图片路径设置给 `original-src`，当页面不可见时，图片不会加载

~~~js
<img original-src="..." />
    
// 通过监听页面滚动，等页面可见时设置图片src
const img = document.querySelector('img')
img.src = img.getAttribute("original-src")
~~~



## 运行时优化



React 应用的性能优化不外乎**减少不必要的重复渲染**。除了**将局部 state 及其依赖的视图拆分为独立的组件**这种方式之外，对于大量全局状态/props 变化引起的渲染，我们依然可以拆分组件，不同之处在于这种拆分的思路是**尽可能降低全局状态/props 变化对其他组件的影响**，以减少不必要的重复渲染。



#### 1.减少重排与重绘



重排：当改变 DOM 元素位置或者大小时， 会导致浏览器重新生成 Render 树， 这个过程叫重排。

重绘：当重新生成渲染树后， 将要将渲染树每个节点绘制到屏幕， 这个过程叫重绘。

针对 React：减少组件渲染次数



实现方式：

1.1 避免 table 布局

1.2 分离读写操作

~~~js
// bad 强制刷新 触发2次重排+重绘
div.style.left = div.offsetLeft + 1 + 'px';
div.style.top = div.offsetTop + 1 + 'px';

// good 缓存布局信息 相当于读写分离 触发1次重排+重绘
var curLeft = div.offsetLeft;
var curTop = div.offsetTop;
div.style.left = curLeft + 1 + 'px';
div.style.top = curTop + 1 + 'px';
~~~

1.3 样式集中改变

~~~js
// 三次重排
div.style.left = '10px';
div.style.top = '10px';
div.style.width = '20px';

// 一次重排
el.style.cssText = 'left: 10px;top: 10px; width: 20px';
~~~

1.4 position 属性设为 absolute 和 fixed



#### 2.避免页面卡顿



浏览器做的工作流程为：解析 HTML => 解析 JS => 解析 CSS => 布局 => 绘制



#### 3.长列表优化



通过虚拟列表，只展示当前用户可视范围的数据，而不是全部。第三方：`react-virtualized`



#### 4.滚动事件性能优化



由于滚动事件发生非常频繁，而频繁地执行监听回调就容易造成 JavaScript 执行与页面渲染之间互相阻塞的情况。

对应滚动这个场景，可以采用**防抖**和**节流**来处理：



防抖：当一个事件频繁触发，而我们希望在事件触发结束一段时间后，才再次触发响应函数时会使用防抖。例如用户一直点击按钮，但你不希望频繁发送请求，你就可以设置当点击后 200ms 内用户不再点击时才发送请求。



节流：当一个事件频繁触发，而我们希望间隔一定的时间再触发相应的函数时， 就可以使用节流来处理。比如判断页面是否滚动到底部。



#### 5.使用 Web Workers



Web Worker 是一个独立的线程（独立的执行环境），这就意味着它可以完全和 UI 线程（主线程）并行地执行 js 代码，从而不会阻塞 UI。它和主线程是通过 onmessage 和 postMessage 接口进行通信的，它使多线程编程称为可能。



#### 6.代码编写



- 使用事件委托：绑定的事件越多， 浏览器内存占得越多，影响性能。利用事件代理的方式就可节省一些内存。
- 当判定条件越来越多时， 倾向于使用 switch，而不是 if-else；
- 布局上使用 flexbox



