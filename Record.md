# Electron



## 本地图片复制到剪切板



~~~js
import { clipboard, nativeImage } from 'electron';
import { app } from '@electron/remote';
const path = require('path');
const fse = require('fs-extra');

// app.getPath -- 数据存储目录
const dirPath = path.join(app.getPath('userData'), 'imageFiles');

// 重要：要先清除剪切板内容 否则数据容易错误
clipboard.clear();

const req = await request(url);

// 判断目录是否存在 不存在自动创建
fse.ensureDir(filePath, async err => {
    const filePath = path.join(dirPath, fileName);
            
    // 判断文件是否存在
    if (!fse.existsSync(finalPath)) {
        // 写入文件
        const out = fse.createWriteStream(filePath);
        await req.pipe(out);
    }
    
    const image = nativeImage.createFromPath(filePath);
    if (!image.isEmpty()) {
        clipboard.writeImage(image);
    }
});
~~~



## 图片转成 base64 复制到粘贴板



~~~js
const img = new Image();
img.crossOrigin = 'Anonymous';
img.src = url;
img.onload = () => {
    let canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    const w = img.width;
    const h = img.height;
    canvas.width = w;
    canvas.height = h;
    ctx.drawImage(img, 0, 0, w, h);

    const base64Image = canvas.toDataURL('image/jpeg', 0.9);

    const image = nativeImage.createFromDataURL(base64Image);
    clipboard.writeImage(image);

    canvas = null;
};
~~~



# JavaScript



## ClipboardItem的坑-只支持png



`ClipboardItem` 的 `type` 支持图片的只有 `image/png`，这就意味着，如果复制的图片是非 png 格式，如 jpg...，就必须先将其转为 png 格式，而转化之后图片的存储占用变大了好几倍

~~~js
try {
    canvas.toBlob(async blob => {
        const copyData = [
            new ClipboardItem({
                [blob.type]: blob
            })
        ];
        await navigator.clipboard.write(copyData).then(() => {
            console.log('复制图片成功');
            successCb && successCb();
        });
    });
} catch (err) {
    console.error(err.name, err.message);
} finally {
    canvas = null;
}
~~~



## Proxy



## 欺骗词法作用域的3种方法，不建议使用



## 利用 DataTransfer  实现



超好用的判断方法

1. DataTransferItem.webkitGetAsEntry()

`webkitGetAsEntry()` 有个属性 `isFile`，我们可以通过它来判断是否为「 纯文件」。

But，兼容性不是很好（现在大多主流浏览器都可使用），实际使用要谨慎，大家可以参考官网 [DataTransferItem.webkitGetAsEntry()](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransferItem/webkitGetAsEntry#specifications) 

~~~js
const { items } = event.dataTransfer;
if (items) {
    for (const item of items) {
        if (
            item.kind === "file" && item.webkitGetAsEntry().isFile
        ) {
            console.log("this is file");
        } else {
            console.log("this is folder");
        }
    }
}
~~~



2. 通过 `FileReader`

思路大概是这样，文件和文件夹不是同一个东西，所以通过 `FileReader` 来读取文件时必然有所不同。比如当文件在进行某些操作时，能正常运行；但如果对象是文件夹，操作就会出错，进而触发 `onerror` 事件，我们可以通过它来判断目标对象是文件还是文件夹。

~~~js
const { files } = event.dataTransfer;
for (const file of files) {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => {
        console.log("this is file");
    };
    reader.onerror = () => {
        console.log("this is folder");
    };
}
~~~



以上方法中，第一种简单、对性能友好，美中不足就是兼容性不好；第二种方法，绝大多数浏览器都能够支持，但如果拖拽的文件很大，文件读取时间就会被拉长，而读取文件的操作又是同步的，容易造成阻塞。

我们不妨可以将两种方法结合一起，如果浏览器支持 `DataTransferItem` 我们使用第一种，否则使用第二种：

~~~js
const { files, items } = e.dataTransfer;
if (items) {
    for (const item of items) {
        if (item.kind === "file" && item.webkitGetAsEntry().isFile) {
            console.log("this is file");
        } else {
            console.log("this is folder");
        }
    }
} else {
    for (const file of files) {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
            console.log("this is file");
        };
        reader.onerror = () => {
            console.log("this is folder");
        };
    }
}
~~~



## 浅谈 DataTransfer  



什么时候会产生

在开发中，涉及到文件上传的场景比比皆是，尤其是在一些 2B、2G 的产品中，为了方便用户操作，许多项目在手动上传的同时还需要支持「 文件拖拽上传」，以此来提高用户体验。说到「 文件拖拽上传」，我们就不得不提到 `DataTransfer` 对象。

`DataTransfer` 对象好比一个使者，他负责携带用户拖拉的文件，并将文件数据信息传递到用户最终拖放的地方。`DataTransfer` 业务范围很广，他可以携带一种或者多种数据，同时也支持一种或者多种的数据类型。除此之外，`DataTransfer` 还有一个很大的权力，他可以修改用户的拖拉数据，并且限制用户的行为，比如限制用户的拖放类型，详情可可以参考 [DataTransfer](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)

因此，了解 `DataTransfer` 是使用「 文件拖拽」的必修课。

用一个例子来了解 DataTransfer  

页面有一个文本输入框，我们可以将输入的文字存到 `DataTransfer` 中，并在蓝色区块拖放置黄色区块的时候，将文字写入黄色框中，而这些事情，都是通过 `DataTransfer` 来完成。

~~~html
 <div class="inputWrap">
     <span>输入你的资料</span>
     <input type="text" id="dragInput" />
</div>
<div class="dragWrap">
    <div draggable="true" class="box box-dragger"></div>
    <div droppable="true" class="box box-dropper"></div>
</div>
~~~



如何实现

当你拖动一个区块（文件）时，最先触发的是 `dragstart` 事件，因此，我们可以在 `dragstart` 触发的时候将输入框文字信息写入到 `DataTransfer ` 中：

~~~js
const dragger = document.querySelector(".box-dragger");
const dragInput = document.querySelector("#dragInput");
let dragTemp;

dragger.addEventListener("dragstart", (e) => {
    dragTemp = e.target;
    // 文字信息写入
});
~~~

在监听拖拽事件的时候要阻止 `dragover` 的默认行为，才能成功地触发 `drop` 事件，将拖拽元素成功放入目标元素中

~~~js
 dropper.addEventListener("dragover", (e) => {
     e.preventDefault();
 });
~~~



dataTransfer.setData

dataTransfer 可以通过 `setData(format, data)` 来写入数据

~~~js
dragger.addEventListener("dragstart", (e) => {
    dragTemp = e.target;
    e.dataTransfer.setData("text/plain", dragInput?.value ?? "");
});
~~~

dataTransfer.getData

dataTransfer 可以利用 `getData(format)` 读取拖拽数据，拖拽事件结束时，会触发 `drop` 事件，我们在这里读取刚才写入的数据，并放入拖放目标里面：

~~~js
const dropper = document.querySelector(".box-dropper"); // 拖放目标
dropper.addEventListener("drop", (e) => {
    // 数据读取
    const dragText = e.dataTransfer.getData("text/plain"); 
    dragTemp.style.margin = "10px";
    dragTemp.style.width = "180px";
    dragTemp.style.height = "180px";
    dragTemp.append(dragText);
    e.target.appendChild(dragTemp);
    e.target.style.color = "#fff";
    e.target.style.background = "#FFC48B";
    dropper.style.border = "none";
});
~~~

对拖拽元素放入目标元素的过程做一些视觉上的交互

~~~js
dropper.addEventListener("dragenter", (e) => {
    dropper.style.background = "#d4965a";
    dropper.style.border = "1px dashed #0a0a0a";
});

dropper.addEventListener("dragleave", () => {
    dropper.style.background = "#FFC48B";
    dropper.style.border = "none";
});
~~~

DataTransfer 对象属性和方法详解

DataTransfer 对象属性

**DataTransfer.dropEffect**

获取当前的拖放操作类型，或者设置当前的拖放操作类型。类型的值可能为： `none`，`copy`， `link` 或 `move`。当设置不同的类型时，拖拽效果会有所不同，主要体现在拖放鼠标的样式上。



**DataTransfer.effectAllowed**

提供可能的所有类型的操作。操作类型可能为：

- `none`：不允许拖拽。鼠标保持禁止状态
- `copy`：允许复制
- `copyLink`：允许复制和链接
- `copyMove`：允许复制和移动操作
- `link`：允许在新位置建立链接
- `linkMove`：允许链接和移动操作
- `move`：允许将元素移动到新位置
- `all` ：允许任意操作
- `uninitialized` ：表示未初始化，效果和 `all` 一样

`effectAllowed` 和 `dropEffect` 的区别：

- `effectAllowed` 主要在 `dragstart` 事件中使用，`dropEffect` 属性主要在 `dragenter` 和 `dragover` 事件中使用
- `effectAllowed` 会限制 `dropEffect` 的值，`dropEffect`  属性只能设置 `effectAllowed` 允许的值；举个简单的例子，如果设置 `effectAllowed` 的值为 `link`，设置 `dropEffect` 的值为 `copy`，两者不一致拖拉是无效的，没法响应 `drop` 事件。不过平时大多数使用场景是拖拉拽，这个属性用的比较少



**DataTransfer.files**

拖拽的本地文件列表。如果拖动操作不涉及文件，则 `files.length` 为 0。除了拖拽，当用户复制文件、粘贴文件也能在 `DataTransfer` 中拿到文件数据。

**DataTransfer.items** *（只读）*

只读属性，用来获取拖拽的数据信息，它是 `DataTransferItem` 类型的数据集合数组。`DataTransferItem` 包含多个属性和方法，属性有 `kind` 和 `type`，方法有 `getAsString()` 和 `getAsFile()`

**DataTransfer.types** *（只读）*

只读属性，由拖拽内容包含的类型组成的数组，我们可以通过遍历来查看拖拽内容包含的所有类型



DataTransfer 对象方法

**DataTransfer.clearData([format])**

对给定类型关联的数据进行删除

**DataTransfer.getData(format)**

返回给定类型的数据，如果该类型的数据不存在或数据传输不包含数据，则返回空字符串

**DataTransfer.setData(format, data)**

设置给定类型的数据。如果该类型的数据不存在，就在末尾添加。如果该类型的数据已存在，则在相同位置把现有数据替换掉

**DataTransfer.setDragImage(img, xOffset, yOffset)**

设置用于拖动的自定义图像，就是拖拽时候我们可以自定义一张图片跟在鼠标后面，属于视觉交互。其中`img `表示自定义图片元素，`offsetX` 表示距离鼠标的水平偏移距离，`offsetY` 表示距离鼠标的垂直偏移距离



## canvas内存限制



**是啥**

`canvas`  是一个可以通过 JavaScript 脚本来绘制图形的 HTML 元素，通常我们可以通过 canvas 来绘制图表、制作构图或者简单的动画。

**我接触到的**

在最近开发工作中，我最经常使用到 `canvas` 的场景是拿到已知的图片 URL，将其转换为 base64 的图片格式，转换过程中需要利用 `canvas.toDataURL()` 方法。

除此之外，我们还可以通过 `canvas.toBlob()` 来创建 `Blob` 对象，这意味着，我们可以利用 `canvas` 将图片 URL 转换为 `Blob` 对象。

**内存限制**

如果你对 `canvas` 并不了解，就很容易忽略 `canvas` 的内存问题。当我看到这篇文章 https://mp.weixin.qq.com/s/hFG1ypsEckVIZb5OCv0H7g，我才知道，如果 `canvas` 创建后没有及时回收，是会占用内存的，占用过多甚至会超出 `canvas` 内存限制，导致图片加载失败。

如下面代码，我们通过数组收集 `canvas` 对象，防止它被垃圾回收，并且声明一个创建 `canvas` 的方法，每个 `canvas` 宽高都为 512px，一个 `canvas` 所占用的内存即为 1MB

...

接下来，我们创建 1000 个 `canvas`，也就是增加 1GB 内存，看看 GPU 的内存占用情况吧~

...

嗯，如你所见，内存占用确实妥妥增加 1GB 的内存，由此可见，`canvas` 对内存确实存在肉眼可见的影响，`canvas` 可存放多少就取决于计算机设备的情况了，只要设备足够 nb，就能吃得下。比如我的笔记本设备是 16GB 的运行内存，当我增加 10GB 的`canvas` 时，页面就开始出现卡顿，这说明此时的计算机面对 10GB 内存的增加显得有些吃力。



那么面对 `canvas` 的内存消耗，我们可以通过什么方法解决？

因为上面的例子中，每次循环都创建了 `canvas` 变量，并且通过数组收集，使其不被回收。JS 会在创建变量时为其自动分配内存，在不使用的时候自动释放内存，释放过程就是「 垃圾回收机制」。

首先我们只创建一个 `canvas`，每次循环在画布上绘制完图像后，对画布进行清理。并且去除掉对 `canvas` 的收集，使其被自动回收，因为在实际开发场景中，在图像绘制完成后我们已经拿到了想要的东西，`canvas` 已经没啥用了，一般不会收集。

修改代码如下：

...

此时，我们再次创建 10GB 的 `canvas` 后可以发现，GPU 的内存占用几乎毫无波澜，并且页面也不会出现卡顿或者崩溃的现象。



**总结**







# React



## 组件懒加载如何实现



## 模拟 willMount

最近，项目要做多语言国际化配置，有这么一个场景：在用户未设置指定的语言环境时，要根据系统来设置语言环境。项目使用的框架为 React + i18next，大家几乎使用 hook 来开发。

总体思路大概是：用户不指定语言则默认跟随系统语言；用户切换语言后重新加载页面。

一开始我是利用 `useEffect` 模拟 `componentDidMount`生命周期，在组件挂载完成后去读取本地语言环境，然后切换到对应语言。获取语言配置主要代码如下：

~~~js
// 创建 i18n class
class LocaleLanguageManager {
    constructor(options = {}) {
        this.options = options;
        i18next.init({
            fallbackLng: 'en-US',
            resources: options.resources
        });
    }
    // ...
}

// i18n 实例
const localeManager = new LocaleManager({
    resources
});
export function getLocaleManager() {
    return localeManager;
}
export function getI18n() {
    return localeManager.getI18n();
}
~~~



更新代码后，会出现这么一个问题，我是在最外层组件（根组件）DOM 挂载完成后去做这个事情，而 React 组件的加载顺序是这样的：`父(组件)beforeCreate -> 父created -> 父beforeMount -> 子beforeCreate -> 子created-> 子beforeMount -> 子mounted -> 父mounted`。

这意味着，需要等到子组件全部加载完成后，才会执行父组件的 `componentDidMount`，也就是说「 切换语言」的动作是在最后执行的，当我语言配置改变后，之前在子组件引用的 `i18n` 语言包并不会得到及时的更新，除非会触发重新加载。

~~~js
useEffect(() => {
    // 切换到对应的语言环境
    changeLanguageByUserLocale(locale)
}, [])
~~~





众所周知，React 生命周期中有一个 `componentWillMount`，不过目前已经被 React 标记为过时了（它在 React 中仍然可以用，但官方不建议使用）。该钩子是在 DOM 挂载之前调用，我们可以利用它在组件初始化渲染之前做一些事情。

But， `componentWillMount` 有一个很明显的缺陷，就是此钩子在页面初始化渲染之前会执行一次或多次，这并不是我们想要的，并且，我们可以利用 `constructor()` 来替代  `componentWillMount`。因此，目前该钩子正慢慢被 React 移除，官方表示，`componentWillMount` 只能继续使用至 React 17。

在 React Hook 中，函数组件并没有 `constructor` 的概念，我们大多数是通过 `useEffect` 来模拟组件的生命周期，比如`componentDidMount`（当组件挂载完成后调用，通常用来监听一些浏览器事件、加载数据） 和 `componentWillUnMount`（在组件即将要）

- `componentDidMount`：当组件挂载完成之后调用，一般用来监听浏览器事件、请求接口加载数据
- `componentWillUnMount`：在组件销毁之前调用，通常是用来做一些清除动作，比如移除事件监听、清除定时器等等

我们可以利用 useEffect 来这么实现这两个生命周期：

~~~js
useEffect(() => {
    // 类似于 componentDidMount
  	window.addEventListener('mousemove', () => {});
    
    return () => {
        // 类似于 componentWillUnMount
        window.removeEventListener('mousemove', () => {})
    }
}, [])
~~~



然而要模拟实现 `componentWillMount`，我们只能另辟蹊径，这里我们通过手写一个 `useComponentWillMount hook`：

~~~js
const useComponentWillMount = (cb) => {
    const willMount = useRef(true)
    if (willMount.current) cb()
    willMount.current = false
}
~~~



值得注意的是，这并不能完全等同于 `componentWillMount`，因为存在代码顺序问题，比如：

~~~js
console.log('111')
useComponentWillMount(() => console.log('222'))
// output:
// 111
// 222
~~~



在这里，`111` 会在 `useComponentWillMount` 之前执行，而在 `class` 的 `componentWillMount` 中，是优先其他代码执行的。

因此，在实际开发中，我们要根据场景和需求，去灵活使用。



## React Hooks 中使用防抖节流要注意这一点



## 拖拽内容是文件还是文件夹



总结坑





# CSS



## 文本超出进行展开/收起



# Webpack



^ 符号

允许次版本号和修订版本号提升，但不允许主版本号提升。简单来说，如果某个依赖包的版本是 ^1.2.1，当你安装依赖包时，可能安装的是 1.3.8 的版本，但不会是 2.x.x 的版本



