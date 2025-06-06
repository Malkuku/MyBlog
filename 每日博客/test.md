1. @Controller（传统 Spring MVC 控制器）
特点

    需要配合 @ResponseBody：默认返回的是 视图名称（View Name），即跳转到某个页面（如 JSP、Thymeleaf 等）。

    适用于传统 Web 应用：比如返回 HTML 页面、JSP 等。

    如果要返回 JSON/XML 数据，必须加 @ResponseBody。

    2. @RestController（RESTful API 专用控制器）
特点

    默认所有方法都带 @ResponseBody：直接返回 JSON/XML 数据，而不是视图。

    适用于 REST API：比如前后端分离项目，返回 JSON 数据。

    不能返回视图，只能返回数据。



这是因为 Electron 的安装过程分为两部分：

    下载 npm 包（这部分已通过淘宝镜像成功完成）

    下载 Electron 二进制文件（这部分仍然尝试从 GitHub 下载）

$ npm install electron --save-dev --electron-mirror=https://npmmirror.com/mirrors/electron/

https://www.electronjs.org/zh/docs/latest/tutorial/installation

https://www.electronjs.org/zh/docs/latest/api/app


为什么这是必要的？

    异步通信本质：

        Electron 的进程间通信是异步的

        invoke() 设计就是返回 Promise 来处理这种异步性

    Promise 链延续：

        如果不 return，Promise 链会在 preload 层中断

        加了 return 才能让 Promise 链延续到渲染进程

    错误传播：

        只有 return 才能将主进程的错误通过 Promise reject 传递到渲染进程

        否则错误会被静默丢弃

实际调用示例对比
能正常工作的版本：
javascript

// preload.js
contextBridge.exposeInMainWorld('myAPI', {
    readFile: () => {
        return ipcRenderer.invoke('file-read') // 有return
    }
})

// 渲染进程
const data = await myAPI.readFile() // 能获取到数据

不能工作的版本：
javascript

// preload.js
contextBridge.exposeInMainWorld('myAPI', {
    readFile: () => {
        ipcRenderer.invoke('file-read') // 无return
    }
})

// 渲染进程
const data = await myAPI.readFile() // data将是undefined

最佳实践建议

    永远 return IPC 调用的结果：

        无论是 invoke() 还是 send() 的响应

    保持异步一致性：

        如果主进程是异步操作，渲染进程也应按异步处理