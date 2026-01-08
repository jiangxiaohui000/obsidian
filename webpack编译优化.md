1，减少解析范围（非常重要）
```js
module.exports = {
  resolve: {
    // 避免全量扫描 node_modules
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],
    // 明确 extensions，避免多次试探
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json']
  }
};
```
2，使用 `cache`（Webpack 5 核心优化）
```js
// 开发 & 生产都开启
module.exports = {
  cache: {
    type: 'filesystem', // 持久化缓存到磁盘
    buildDependencies: {
      config: [__filename] // 配置变更时失效
    }
  }
};
```
3，缩小 Loader 作用范围
```js
{
  test: /\.tsx?$/,
  include: path.resolve(__dirname, 'src'),
  use: 'babel-loader'
}
```
4，启用 `loader` 缓存
```js
{
  loader: 'babel-loader',
  options: {
    cacheDirectory: true // 缓存转换结果
  }
}
```
5，使用更快的编译器（本质性优化）
用swc-loader / esbuild-loader代替Babel，快10-100倍
更快的压缩工具`esbuild`（Rust）比 Terser 快 10～100 倍。插件：`esbuild-webpack-plugin`
6，配置项区分开发环境和生产环境
开发环境：
使用 `eval-source-map` 或 `eval-cheap-module-source-map`
```js
module.exports = {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map' // 最快的 source map
};
```
关闭不必要的插件
- 开发环境不需要 `MiniCssExtractPlugin`、`TerserPlugin` 等
- 用 `style-loader` 代替 CSS 文件提取
使用 `webpack-dev-server` 的 `lazy` / `hot` 优化
```js
devServer: {
  hot: true,        // 热更新（只更新模块）
  liveReload: false // 关闭整页刷新
}
```
生产环境：
代码分割（SplitChunks）避免重复打包
```js
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        chunks: 'all'
      }
    }
  }
}
```
并行压缩（Terser）
```js
const TerserPlugin = require('terser-webpack-plugin'); 
optimization: { 
	minimizer: [ 
		new TerserPlugin({ 
			parallel: true, // 多进程压缩 
			terserOptions: { 
				compress: { 
					drop_console: true 
				} // 移除 console 
			} 
		})
	] 
}
```
生产环境的source map使用hidden-source-map：
```js
// webpack.prod.js
module.exports = {
  devtool: 'hidden-source-map',
  optimization: {
    minimize: true
  }
};
```
行为说明
-  生成 `.map` 文件
-  浏览器 **不会**加载
-  将map文件上传到错误监控平台
-  不暴露源码
-  仍可用于错误反解