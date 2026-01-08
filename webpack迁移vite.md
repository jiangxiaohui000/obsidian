可以使用工具：https://github.com/originjs/webpack-to-vite
在不修改源码的基础上，配置vite
1，安装vite依赖：
```json
yarn add vite @vitejs/plugin-vue vite-plugin-html -D
```
2，为了不影响线上打包，新建一个模版文件index.vite.html，将index.html的内容粘进去
3，将index.vite.html文件放在项目根目录，不能放在public文件夹里了
4，新增vite.config.ts配置文件
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { createHtmlPlugin } from 'vite-plugin-html';
import EnvironmentPlugin from 'vite-plugin-environment';
import { viteCommonjs } from '@originjs/vite-plugin-commonjs'

export default defineConfig({
	// 项目启动遇到这个报错：ReferenceError: process is not defined
	// 因为 webpack 启动时会根据 node 环境将代码中的 `process` 变量会将值给替换，而 vite 未替换该变量，所以在浏览器环境下会报错。
	define: { 
		'process.env': { 
			NODE_ENV: JSON.stringify(process.env.NODE_ENV), // `import.meta.env` 是一个对象，不能直接赋值给 `NODE_ENV`
			// 可以加更多...
		} 
	}
	plugins: [
		vue(),
		// 代替webpack中htmlwebpackplugin
		createHtmlPlugin({
			minify: true, // 压缩 HTML
			entry: 'src/main.ts',
			template: 'public/index.vite.html', 
			inject: { 
				// 注入到 index.html 的数据 
				data: { 
					title: '我的', 
					// 高级用法
					title: process.env.NODE_ENV === 'production' ? '生产环境' : '开发环境',
					description: '介绍', 
				}, 
				// 可选：注入内联 script 
				tags: [ 
					{ 
						tag: 'script', 
						children: `window.GLOBAL_CONFIG = { apiBase: '${process.env.VITE_API_BASE}' };`, 
					} 
				] 
			},
		}),
		// 如果项目中在.env文件中设置了以 `VUE_APP_XXX` 开头的环境变量，然后就可以继续使用process.env.VUE_APP_API_BASE
		EnvironmentPlugin(
			{ 
				// 将 VITE_ 变量映射到 process.env 
				VUE_APP_API_BASE: 'VITE_API_BASE', 
				VUE_APP_TITLE: 'VITE_TITLE', 
			}, 
			{ 
				prefix: '' // 不加 VITE_ 前缀
			}
		),
		// 项目中通过 `require()` 引入了图片，
		// webpack 支持 commonjs 语法，而 vite 开发环境是 esmodule 不支持 require。
		// 可以通过 `@originjs/vite-plugin-commonjs` 插件，它能解析 `require` 进行语法转换以支持同样效果
		viteCommonjs(),
	],
	// 如果项目在 less 文件中定义了变量，并在 webpack 的配置中通过 `style-resources-loader` 将其设置为了全局变量。
	// 我们可以在 `vite.config.js` 中添加如下配置引入文件将其设置为全局变量
	css: {
		preprocessorOptions: {
			less: {
				// 注入全局变量，无需手动 @import 
				additionalData: `@import "@/styles/variables.less";`,
				javascriptEnabled: true, // 如果用了 Less 函数（如 tint()），需开启 
			}, 
		}, 
	},
	resolve: {
	    alias: {
		// alias 对齐，不然会导致引用报错
	    '@': path.resolve(__dirname, './src'),
	    '~@': path.resolve(__dirname, './src'), // vite 中 css 等样式文件的 alias 不需要加 `~` 前缀，所以也需要配置下 `~@`
		}
	}
})
```
插件讲解：
**vite-plugin-html:**
用于在 Vite 项目中动态注入数据到 HTML 模板中的插件。
让 Vite 的 `index.html` 支持动态数据注入、环境变量替换、自定义 meta 和脚本插入，弥补 Vite 默认 HTML 静态模板的不足。
```html
<!DOCTYPE html>
<html lang="">
  <head>
-   <title><%= htmlWebpackPlugin.options.title %></title> // 这是webpack的用法
+   <title><%= title %></title> // 这是vite的用法
  </head>
</html>
```