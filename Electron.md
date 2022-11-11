# Electron 实战



## 生态

- electron-builder：Electron 构建工具
- Vue CLI Plugin Electron Builder：结合 Vue
- electron-vue：结合 Vue
- electron-react-boilerplate：结合 React
- angular-electron：结合 angular
- awesome-electron：Electron 相关项目



## 基础



启动 script 脚本运行前会自动新建一个命令行环境，把 node_modules/.bin 加入系统环境变量中，接着执行 scripts 指定的脚本内容，执行完成后再把 node_modules/.bin 从系统环境变量中删除。因此，当前目录下的 node_modules/.bin 子目录 里面的所有脚本，都可以直接用脚本名调用，不必加上路径

JavaScript 是不支持模块化的，Node.js 使用的是 CommonJS 规范，所以在 electron 中可以使用 `require` 来引入模块

如果 `loadFile` 加载一个网络页面，你无法验证页面内容是否安全，应关闭选项：`webPreferences: {nodeIntegration:false}`



## 主进程和渲染进程



### 区分



主进程负责监听应用程序的生命周期事件、 启动第一个窗口、加载 index.html 页面、应用程序关闭后回收资源、退出程序等工作渲染进程负责完成渲染界面、接收用户输入、响应用 户的交互等工作

一个 Electron 应用只有一个主进程，但可以有多个渲染进程。一个 `BrowserWindow` 实例就代表一个渲染进程。当 `BrowserWindow`实例被销毁后，渲染进程也跟着终结

在 Electron 中，GUI 相关模块仅在主进程中可用。如果想在渲染进程中做主进程才能干的事，可以让渲染进程通知主进程来做



### 调试

**主进程调试配置**

~~~json
// launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "调试主进程",
            "type": "node",
            "request": "launch",
            "cwd": "${workspaceRoot}",
            "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron",
            "windows": {
                "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron.cmd"
            },
            "args": ["."],
            "outputCapture": "std"
        }
    ]
}
~~~



**调试渲染进程**

- 使用 Chrome 浏览器的开发者工具
- 在 `loadFile` 后自动开启调试状态 `win.webContents.openDevTools();`



### 进程互访



#### 渲染进程访问主进程

通过 electron 的 `remote` 模块，现在已被废弃，可使用第三方包 `@electron/remote`

remote 对象的属性和方法都是主进程的属性和方法的映射。通过 remote 访问时，Electron 内部会构造一个消息，这个消息从渲染进程传递给主进程，主进程完成相应的操作，把操作结果以远程对象的形式返回给渲染进程



#### **主进程访问渲染进程**

渲染进程是主进程创建的，访问方便，如控制窗口加载 url： `winObj.webContents.loadURL()`



### 进程间消息传递



#### 渲染进程-主进程

~~~js
// 渲染进程
ipcRenderer.send(
    "channel_name",
    { name: "param1" },
    { name: "param2" }
);
// 主进程
ipcMain.on('channel_name', (event, param1, param2) => {
    console.log(param1, param2, event.sender); // sender: 渲染进程webContents对象实例
});
~~~

通信过程中消息发送的 json 对象会被序列化和反序列化，所以 json 对象中包含的**方法**和原型链上的数据不会被传送

如果主进程在多处监听同一管道的代码，当该管道有消息发来时，多处的监听事件都会被触发

如果消息需要主进程同步处理，可通过 `ipcRenderer.sendSync` 方式发送，但同步方式会阻塞进程，最好用异步方式



#### 主进程-渲染进程

~~~js
// 主进程
ipcMain.on("renderer_to_main", (event, param1, param2) => {
    win.webContents.send("main_to_renderer", param1, param2);
});
// 渲染进程
ipcRenderer.on("main_to_renderer", (event, param1, param2) => {
    console.log(param1, param2, event.sender);
});
~~~

一旦主进程发来消息， 当前页面所有的消息监听函数都会被触发。如果打开新窗口，并加载同样页面，设置同样的监听函数，当主进程再发送的消息时，新窗口却不会触发监听事件。因为我们使用 `win.webContents.send` 与渲染进程通信，这意味着 Electron 只给 win 所代表的渲染进程发送消息

如果在多个窗口的渲染进程给主进程发送相同消息，发送完成后，需要主进程响应，也就是发送消息给对应的渲染进程，该如何处理：主进程接收消息事件的 event.sender 代表发送者的 `webContents`，我们可以通过此对象来给对应的窗口发消息

~~~js
ipcMain.on('msg_render2main', (event, param1, param2) => {
    event.sender.send('msg_main2render', param1, param2)
});
~~~

也可以使用 event.reply

~~~js
ipcMain.on('msg_render2main', (event, param1, param2) => {
    event.reply('msg_main2render', param1, param2)
});
~~~

如果渲染进程传递的是同步消息，可以直接设置返回 returnValue 属性响应给渲染进程。这种方式不需要监听，当消息发送调用成功时，返回值就是主进程设置的 `event.returnValue` 的值

~~~js
let returnValue = ipcRenderer.sendSync('msg_render2main', { name: 'param1' });
~~~



#### 渲染进程之间消息传递



场景：多个窗口（渲染进程）之间相互传递消息，通常是通过主进程中转消息来完成窗口之间的消息传递

如果你知道消息接收窗口的 webContents 的 id，可以这么做：

~~~js
// send 第1个参数为窗口 id，第2参数为管道名称
ipcRenderer.sendTo(win2.webContents.id, 'channel_name', { name: 'param1' })
// receive
ipcRenderer.on('channel_name', (event, param1, param2) => {
    console.log(param1, param2, event.sender);
});
~~~



### remote模块的局限性



1. 性能消耗大：remote 模块可访问主进程的对象、类型、方 法，但这些操作都是跨进程的，跨进程操作在性能上的损耗可能是进程内操作的几百倍甚至上千倍
2. 制造混乱：使用主进程的某个对象，可能会在某个时刻触发事件，导致错误发生，且难以排查
3. 制造假象：在渲染进程中的 remote 模块是主进程的对象的映射，即代理对象，原型链上的属性并不会映射到代理对象上
4. 安全问题：恶意代码可能通过原型污染攻击来模拟 remote 模块的远程消息，来获取访问主进程模块的权力



这也是 remote 模块被 electron 移除的原因



## 框架引入



node 常用的全局变量

- __dirname：当前所在目录的绝对路径
- __filename：当前运行脚本的绝对路径
- global：node 所在的全局环境，相当于 window 对象
- process：node 内置模块，可用于操作当前进程
- console：node 内置模块，提供命令行环境的输入输出



## 窗口



### 窗口常用属性



- 窗口位置：`x y center movable`

  说明：通过 x y 可以控制窗口位置，如果没有设置默认显示在屏幕正中

- 窗口大小：`width height resizable minizable maximizable ` 

- 窗口边框、标题栏与菜单栏属性 `title icon iframe autoHideMenuBar titleBarStyle`

- 控制渲染进程访问 node 环境能力：`nodeIntegration nodeIntegrationInWorker nodeIntegrationInSubFrames `

- 开发者控制渲染进程加载的页面：`preload webSecurity contextIsolation`





### 窗口标题栏和边框



- 设置 `frame: false` 隐藏窗口的边框和标题栏

- 用户拖拽移动窗口设置 `-webkit-app-region: drag`，如果不希望某个子元素有拖拽功能，可对其设置 `no-drag`

- 窗口控制 `getCurrentWindow()`：关闭-close、最小化-minimize、最大化-maximize、恢复原始尺寸-restore

- 如果只监听 `maximize` 和 `unmaximize` 事件去控制窗口，那么如果刷新页面会有问题

- 无论是防抖还是节流，其主要作用都是防止在短期内频繁调用某函数，导致大量无效操作，损耗系统性能。如：在应用的拖拽移动、放大缩小的场景中

- `localstorage` 记录和恢复窗口状态
- 窗口位置恢复会抖动，先显示在正中间再移动到正确位置，可通过 `show` 先隐藏，恢复位置后展示来解决




### 不规则窗口



创建不规则窗口：先把窗口设置为透明，再通过 dom 元素去控制窗口形状

~~~js 
win = new BrowserWindow({
    width: 380,
    height: 380,
    transparent: true,
    frame: false,
    resizable: false, // 透明的窗口不可调整大小
    maximizable: false,
    //...
})
~~~



通过 `setIgnoreMouseEvents` 方法去击穿透明区域



### 窗口控制



阻止窗口关闭

1. 可以通过 `onbeforeunload` 来阻止，在该事件中操作 dom，如添加浮动 div 来提示关闭窗口。注意，此处不能调用 `win.close` 来关闭，因为 `close` 又会触发 `onbeforeunload` ，导致无法关闭

2. `close` 事件：`win.on('close', e => { e.preventDefault() })`，在事件内通知渲染进程，让其显示提示性信息



### 多窗口竞争资源



场景：多个渲染进程同时读写同一个本地文件

解决方案：

1. 渲染进程之间通信，对文件操作锁定/释放，保证读写操作有序进行
2. 利用 Node 的 `fs.watch` 监视文件变化，保证当前文件内容是最新的，文件的读写权力交给主进程
3. 在主进程设置令牌 `global.fileLock`，在渲染进程读取该令牌获取文件的读写权力



### 模态窗口与父子窗口



模态窗口：一旦模态窗口打开，用户就只能操作该窗口，而不能再操作其父窗口。此时，父窗口处于禁用状态，只有等待子窗口关闭后，才能再操作其父窗口

~~~js
this.win = new remote.BrowserWindow({
    parent: remote.getCurrentWindow(),
    modal: true,
    webPreferences: {
        nodeIntegration: true
    }
});
~~~



### MAC 系统关注点



获取当前操作系统

- `process.platform`
  - 值为 `darwin ` 表示当前操作系统为 Mac 系统
  - 值为 `win32` 时，表示当前操作系统为 Windows 系统（不管是不是64位的）
  - 值为 `linux` 时，表示当前操作系统为 Linux 系统
  - 此外， 其还可能是其他值，但在 Electron 应用中不常用
- `process.platform`

- `require('os').platform()`



## 界面



### 页面内容



#### **获取 webContents 实例**

从已拥有的窗口对象获取：`win.webContents`

在主进程获取当前激活状态下的窗口：`require('electron').webContents.getFocusedWebContents();`

从渲染进程获取当前窗口的 webContents 实例：`remote.getCurrentWebContents();`

通过 id 获取窗口的 webContents 实例：`require('electron').webContents.fromId(yourId)`

遍历窗口筛选获取：`for (let webContent of  webContents.getAllWebContents()) {  }`



**BrowserWindow 对象相关方法**

`BrowserWindow.getFocusedWindow()` 获取当前激活状态的窗口

`BrowserWindow.fromId(id)` 根据窗口ID获取窗口实例；

`BrowserWindow.getAllWindows()` 获取所有窗口



#### **页面加载事件及触发顺序**



1. ​	 页面加载过程的第一个事件
2. page-title-updated 页面标题更新事件
3. dom-ready DOM 加载完成时触发
4. did-frame-finish-load 框架加载完成时触发
5. did-finish-load 当前页面加载完成时触发
6. page-favicon-updated 页面 icon 图标更新时触发
7. did-stop-loading 所有内容加载完成时触发



#### 页面跳转事件



#### 单页应用页面跳转



#### 页面缩放



`setZoomFactor` 设置页面的缩放比例

`getZoomFactor` 获取当前网页的缩放比例

`webContents` 的 `setZoomLevel` 可设置网页缩放等级



#### 渲染海量数据



使用 `canvas` 技术在画布上渲染大量数据元素



### 页面容器



场景：需要在窗口内包含其他子页面



#### webFrame

- 每有一个 iframe 就对应着一个 webFrame 实例 
- 使用： `const { webFrame } = require('electron');`
- 相同域下才能使用实例的一些方法
- 不能单独缩放
- 创建窗口时设置 `nodeIntegrationInSubFrames`，子页面集成 node 环境



#### webview

- webview 是 Electron 独有标签，开发者可通过标签在网页中嵌入另外一个网页的内容
- 它与普通的 DOM 标签并没有太大区别，也可设置 id 和 style 属性
- webview 默认不可用，如果要用此标签，需在创建窗口时，需要设置 `webviewTag:true`
- webview 加载第三方内容可能有安全风险，未来可能被删除，官方不推荐使用



#### BrowserView

- BrowserView 被设计成一个子窗口的形式，依托于 BrowserWindow，可绑定到 BrowserWindow 具体的区域，随 BrowserWindow 的放大/缩小而放大/缩小、移动而移动。看起来就像是 BrowserWindow 里的一个元素
- 显示和隐藏可以通过 `removeBrowserView` 和 `addBrowserView`，或者通过 CSS 隐藏
- 官方推荐



### 脚本注入

开发者可以把一段 JavaScript 代码注入到目标网页中。适用于需要注入大量业务逻辑到第三方网站中的场景



preload

~~~js
let win = new BrowserWindow({
    webPreferences: { 
        preload: yourJsFilePath,
        nodeIntegration: true 
    }
});
~~~

无论页面是否开启 `webPreferences.nodeIntegration`，注入的脚本都有能力访问 Node.js 的 API。若开启，第三方网页也有访问 nodejs 的权利；若关闭，只有注入的脚本才有权力



executeJavaScript

适合注入少量逻辑代码的场景

~~~js
win.once('did-finish-load', async () => {
    let result = await win.webContents.executeJavaScript("document.querySelector('img').src");
    console.log(result);
})
// 注入 CSS
let key = await win.webContents.insertCSS("html, body { background-color: #f00!important; }");
~~~



禁用 beforeunload

推荐方案：

~~~js
win.webContents.on('will-prevent-unload', event => {
    event.preventDefault();
});
~~~



### 页面动画



## 数据



### 用户数据目录

`app.getPath("userData")` 会根据操作系统返回对应地址



读写本地文件 

~~~js
// 写
let dataPath = app.getPath("userData");
dataPath = path.join(dataPath, "a.data");
fs.writeFileSync(dataPath, yourUserData, { encoding: 'utf8' })
// 读
fs.readFileSync(dataPath,{ encoding: 'utf8' });
~~~



推荐 `fs-extra` 第三方库

`lowdb`  `fs-extra`  `electron-store`



浏览器技术持久化



在开发 Electron 应用时，推荐使用 Cookie 和 IndexedDB 来存储数据



Electron 提供了专门用来读取 Cookie 的 API

~~~js
//获取Cookie
let getCookie = async function(name) {
    let cookies = await remote.session.defaultSession.cookies.get({name});
    if(cookies.length>0) return cookies[0].value;
    else return '';
}
//设置Cookie
let setCookie = async function(cookie) {
    await remote.session.defaultSession.cookies.set(cookie);
}
~~~

为了防止 XSS 脚本攻击，可以给 Cookie 设置 `HttpOnly` 属性



### 清空浏览器缓存



Electron 提供了一个简洁的清除方式

~~~js
await remote.session.defaultSession.clearStorageData({ 
    storages: 'cookies,localstorage' 
})
// session 值可以为
appcache,cookies,filesystem,indexdb,localstorage,shadercache,websql,serviceworkers,cachestorage
~~~



### 使用SQLite持久化数据





## 系统



### 系统对话框



渲染进程中创建文件打开对话框

~~~js
const { dialog, app } = require("electron").remote;
let filePath = await dialog.showOpenDialog({
    title: "我需要打开一个文件",
    buttonLabel: "按此打开文件",
    defaultPath: app.getPath('pictures'),
    properties:"multiSelections",
    filters: [
        { name: "图片", extensions: ["jpg", "png", "gif"] },
        { name: "视频", extensions: ["mkv", "avi", "mp4"] }
    ]
});
~~~



「 关于」对话框

~~~js
// 指定内容
app.setAboutPanelOptions({
    applicationName:'redredstar',
    applicationVersion:app.getVersion(),
    copyright:''
})
~~~



### 菜单



#### **窗口菜单**

Windows 窗口菜单显示与隐藏设置，但该方法不靠谱

~~~js
let win = new BrowserWindow({
    webPreferences: { nodeIntegration: true },
    autoHideMenuBar:true // 隐藏窗口的系统菜单
});
~~~



通常创建自己的系统菜单来覆盖 electron 菜单

~~~js
let Menu = require('electron').Menu;
let templateArr = [{}];
let menu = Menu.buildFromTemplate(templateArr);
Menu.setApplicationMenu(menu);
~~~



在 Mac 操作系统下，第一个菜单的位置要留出来，因为 Mac 操作系统会自动帮应用设置第一个菜单

~~~js
if(process.platform === 'darwin'){
    templateArr.unshift({label:''})
}
~~~



#### HTML右键菜单

- 用 HTML + CSS + JS 实现菜单组件
- 缺陷：只能显示在窗口内部，不能浮于窗口之上



#### 系统右键菜单

可浮于窗口之上

~~~js
let {Menu} = require('electron').remote;
let menu = Menu.buildFromTemplate([]);
window.oncontextmenu = function(e) {
    e.preventDefault();
    menu.popup();
};
~~~



#### 自定义系统右键菜单

使用系统右键菜单有个不足，即开发者很难对菜单样式定制化，想达到这个目的，可以在鼠标右键时在鼠标定位旁显示一个无边框窗口

窗口的坐标应是相对于屏幕的，所以不可取 oncontextmenu 事件内的 e.clientX 和 e.clientY 作为坐标

~~~js
window.oncontextmenu = function(e) {
    const { screen } = require("electron").remote;
    let point = screen.getCursorScreenPoint(); // 此为鼠标相对于屏幕的位
    console.log(e.clientX + "," + e.clientY);
}
~~~

控制菜单窗口的显隐状态，而不是频繁创建窗口



### 快捷键



监听网页按键事件，只有在窗口激活的时候才可用

~~~js
window.onkeydown = function(e) {
    if ((e.ctrlKey || e.metaKey) && e.keyCode == 83) {
        alert("按键监听");
    }
};
~~~



监听全局按键事件

~~~js
const { globalShortcut } = require('electron')
globalShortcut.register('CommandOrControl+K', () => {
    console.log('按键监听')
})
~~~

如果你注册的快捷键已经被另外一个应用注册过了，你的应用将无法注册此快捷键，也不会收到任何错误通知



### 托盘图标



**注册托盘图标**

~~~js
let { app, BrowserWindow, Tray } = require('electron');
let path = require('path');
let tray;
app.on('ready', () => {
    let iconPath = path.join(__dirname, 'icon.png');
    tray = new Tray(iconPath)
}
~~~



**托盘图标闪烁**

每隔 n 毫秒切换图标来实现闪烁

~~~js
let flag = true;
setInterval(() => {
    if (flag) {
        tray.setImage(iconPath2);
        flag = false;
    } else {
        tray.setImage(iconPath);
        flag = true;
    }
}, 600)
~~~



**托盘图标菜单**

- Electron 双击托盘图标，除了会触发 `double-click` 事件，也会触发 `click` 事件
- 鼠标右键单击托盘图标，则只会触发 `right-click` 事件不会触发 `click` 事件
- 一旦为托盘图标注册了菜单，托盘将不再响应你注册的 `right-click` 事件

~~~js
let { Tray,Menu } = require('electron');
let menu = Menu.buildFromTemplate([{
    click() { win.show(); },
    label: '显示窗口',
    type: 'normal'
}, {
    click() { app.quit(); },
    label: '退出应用',
    type: 'normal'
}]);
tray.setContextMenu(menu);
~~~



### 剪切板



图片写入剪切板

~~~js
let imgPath = path.join(__static, "icon.png");
let img = nativeImage.createFromPath(imgPath);
clipboard.writeImage(img);
// 清除剪切板数据
clipboard.clear()
~~~



读取剪切板图片

~~~js
let img = clipboard.readImage();
let dataUrl = img.toDataURL();
~~~



### 系统通知



开发网页时，使用系统通知要先获得用户授权

Notification 是一个 HTML5 的 API，在 Electron 渲染进程中也可以自由使用，且不需要用户授权

~~~js
Notification.requestPermission(function (status) {
    if (status === "granted") {
        let notification = new Notification('您收到新的消息', {
            body: '此为消息的正文'
        });
    } else {
        // 用户拒绝授权
    }
});
~~~



主进程发送系统通知：通过 remote 模块访问主进程中 Notification 的类型

~~~js
const { Notification } = require("electron").remote;
let notification = new Notification({
    title:"您收到新的消息",
    body: "此为消息的正文，点击查看消息",
});
notification.show();
notification.on("click", function() {
    alert("用户点击了系统消息");
})
~~~



### 其他

- 使用系统默认应用打开文件
- 接收拖拽到窗口的文件
- 使用系统字体
- 最近打开的文件



## 调测



### 测试



- 单元测试：`mocha`
- 界面测试：`Spectron`



### 调试



渲染进程性能追踪：开发工具-performance



自动追踪性能问题：`contentTracing` 模块，拿到日志在 `chrome://tracing` 分析

~~~js
// 6秒性能监控
(async() => {
    const { contentTracing } = require('electron');
    await contentTracing.startRecording({
        include_categories: ['*']
    });
    await new Promise(resolve => setTimeout(resolve, 6000));
    const path = await contentTracing.stopRecording()
    console.log('性能追踪日志地址：' + path);
    createWindow();
})()
~~~



### 性能优化技巧



1. 引入第三方模块需谨慎
2. 尽量避免等待：
   1. 对耗时的计算工作，应避免使用同步方法
   2. 大量耗时任务考虑使用 Web Worker 来处理
3. 尽量合并资源：
   1. 开发Web应用时，我们会把大文件拆分成小文件，加快下载速度。但 Electron 是本地应用，且资源在客户电脑本地环境下的，下载速度已不是问题
   2. 应尽可能把 CSS 和 JavaScript 合并到同一个文件，避免不必要的 require 和浏览器加载工作



### 调试工具



开发环境： `yarn add devtron --dev`

生产环境：https://github.com/bytedance/debugtron



### 日志



**业务日志**

`yarn add electron-log`



**网络日志**

开发者工具-请求数据



**崩溃报告**

~~~js
let { crashReporter } = require('electron')
crashReporter.start({
    productName: 'YourName',
    companyName: 'YourCompany',
    submitURL: 'http:// localhost:8989/',
    uploadToServer: true
});
~~~

