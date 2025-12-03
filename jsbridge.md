JSBridge（JavaScript Bridge）是一种用于在前端（Web页面）和客户端（Native应用）之间进行通信的技术。它的原理基于在 WebView 或 Web 容器中注入一个 JavaScript 接口，使得前端 JavaScript 能够调用客户端原生的 API，同时客户端也能够通过 WebView 向前端发送数据。

**主要原理如下：**

1.  **注入接口：** Native 应用在加载 WebView 时，会注入一个 JavaScript 接口到 WebView 的上下文中。这个接口包含了可以由前端调用的原生方法。
2.  **调用原生方法：** 在前端 JavaScript 中，可以通过调用这个注入的接口来触发客户端原生方法的执行。这些方法通常用于执行一些在客户端环境中可用但在 Web 中不可用的功能，比如获取设备信息、调用摄像头等。
3.  **消息传递：** 前端和客户端之间通过一定的协议进行消息传递。前端通过调用接口触发原生方法，而客户端通过回调函数或其他方式将结果传递回前端。
- **H5调用Native**：原生将webviewjavascriptbridge绑定在window上，H5直接调用这个对象中的原生接收方法。
- **Native调用H5**：H5将jsbridge绑定在window上，Native通过原生方法调用这个对象上的H5接收方法。

JSBridge 简单来讲，主要是 **给 JavaScript 提供调用 Native 功能的接口**。它的核心是 **构建 Native 和非 Native 间消息通信的通道**，而且是 **双向通信的通道**。

JavaScript 调用 Native 的方式，主要有两种：**注入 API** 和 **拦截 URL SCHEME**。

**注入 API** 方式的主要原理是，通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。

**拦截 URL SCHEME** 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。
**缺陷**：会有 url 长度的隐患。**优势**：支持 iOS6

JavaScript 调用 Native 推荐使用 **注入 API** 的方式。
Native 调用 JavaScript 则直接执行拼接好的 JavaScript 代码即可。


### 为什么webview会很慢？
客户端需要先花费时间初始化WebView完成后，才开始加载。