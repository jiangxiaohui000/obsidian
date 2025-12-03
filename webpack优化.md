**分析工具：** `webpack-bundle-analyzer` `speed-measure-webpack-plugin`

**优化手段：**
1. Js压缩  `terser-webpack-plugin`
2. Css压缩 `optimize-css-assets-webpack-plugin` `mini-css-extract-plugin`
3. 图片压缩  `image-webpack-loader`
4. 代码拆分 `splitChunksPlugin`
```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 20000,
      minRemainingSize: 0,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,
      cacheGroups: {
        defaultVendors: {
          test: /[\/]node_modules[\/]/,
          priority: -10,
          reuseExistingChunk: true,
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```
5. 减少查找过程，缩小打包作用域
- 使用 `resolve.extensions` 或者 导入模块时，尽量带上文件后缀名。
- 使用 `resolve.alias` 减少查找过程
- 使用 `resolve.modules` 配置 `webpack` 去哪些目录下寻找第三方模块，默认情况下，只会去 `node_modules` 下寻找，如果你我们项目中某个文件夹下的模块经常被导入，不希望写很长的路径，那么就可以通过配置 `resolve.modules` 来简化。
```js
module.exports = {
  resolve: {
    modules: ['./src/components', 'node_modules']
  }
}
```
- 缩小构建目标，借助 `include` 和 `exclude` 这两个参数，规定 loader 只在那些模块应用和在哪些模块不应用
6. 多进程打包 `thread-loader`  webpack 每次解析一个模块，thread-loader 会将它及它的依赖分配给 worker 线程中，从而达到多进程打包的目的
7. 缓存 
- cache-loader 
- 或者开启相应 loader 或者 plugin 的缓存
- 或者 hard-source-webpack-plugin 来提升二次构建的速度
8. `ES6 Modules + Tree-Shaking`
9. 抽离第三方模块
10. `externals` 配置选项提供了「从输出的 bundle 中排除依赖」的方法, **防止**将某些 `import` 的包**打包**到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖。例如，从 CDN 引入，而不是把它打包

**构建流程：**
1. 初始化参数
2. compiler对象开始编译
3. entry确定入口文件
4. 模块编译：递归查找依赖，调用loader完成模块编译
5. 根据模块依赖关系把多个模块组装成chunk输出
6. 根据output确定输出路径和文件名

**简单说**
- 初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler
- 编译：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理
- 输出：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中

**文件指纹**
- 打包后输出的文件名的后缀
- Hash: 以项目为单位，项目内容改变了，则会生成新的hash，内容不变则hash不变
- Chunkhash：以chunk为单位，当一个文件内容改变，则整个chunk组的模块hash都会改变
- Contenthash：以自身内容为单位，依赖不算

**编写一个loader**

```js
module.exports = function (soure) { 
  return source.replace('hello webpack', "你好呀，webpack！") 
}
```
- loader是一个函数
- 不能使用箭头函数，因为要用到上下文的this
- soure接收到的是待处理的文件源码
- return处理后的文件源码，也是下一个loader接收到的文件源码

**实现一个plugin**
- 自定义plugin是一个构造函数

```js
class HtmlPlugin { 
  constructor (options) {
  
  } 
  apply(compiler) { 
    compiler.hooks.emit.tap('HtmlPlugin'，compilation => { 
      const content = '<html><body>fake html</body></html>' ;
      compilation.assets['fake.html'] = { 
        source: function () {
          return content;
        }, 
        size: function () {
          return content.length;
        } 
        } 
      })
    }
}
module.exports = HtmlPlugin
```
-   `compiler` 对象包含了`Webpack` 环境所有的的配置信息。这个对象在启动 `webpack` 时被一次性建立，并配置好所有可操作的设置，包括 `options`，`loader` 和 `plugin`。当在 `webpack` 环境中应用一个插件时，插件将收到此 `compiler` 对象的引用。可以使用它来访问 `webpack` 的主环境。

-   `compilation`对象包含了当前的模块资源、编译生成资源、变化的文件等。当运行`webpack` 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 `compilation`，从而生成一组新的编译资源。`compilation` 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用。
-   `compiler`代表了整个webpack从启动到关闭的生命周期，而`compilation` 只是代表了一次新的编译过程

**热模块替换**
1. 首先，将之前的MiniCssExtractPlugin.loader替换回style-loader 
2. 给devServer中加上hot: true 
3. 引入webpack: const Webpack = require('webpack') 
4. 在plugin中添加：new Webpack.HotModuleReplacementPlugin()

**按需加载**

```js
document.getElementById('btn').onclick = function() {
    import('./handle').then(fn => fn.default());
}
```
`import()` 语法，需要 `@babel/plugin-syntax-dynamic-import` 的插件支持，但是因为当前 `@babel/preset-env` 预设中已经包含了 `@babel/plugin-syntax-dynamic-import`，因此我们不需要再单独安装和配置。
