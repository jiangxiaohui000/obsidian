项目中的使用场景：缓存API接口数据，例如城市接口数据，配置项接口数据。
流量广告这个项目，使用Service Worker使得页面二次打开的白屏时间减少50%，弱网环境下可以达到80%。

使用**WorkBox**  webpack插件。

Workbox 提供了两种模式：
1. **GenerateSW**：全自动模式。适合大多数简单的缓存需求，自动生成 SW 文件。
2. **InjectManifest**：自定义模式。如果你需要自己写 SW 逻辑（如处理推送通知），用这个。

只缓存GET请求。

```js
npm install workbox-webpack-plugin --save-dev
```

属性讲解：
###### cacheId VS cacheName：
`cacheId` 的作用是为 Workbox 生成的所有 CacheStorage 缓存名称添加一个统一的前缀，用于区分不同站点、不同环境或不同应用，避免缓存冲突。
cacheId 是“命名空间”，cacheName 是“具体容器名”。
在同域多应用、微前端或多环境部署的场景下，如果不配置 cacheId，不同 Service Worker 可能会操作同一批缓存，从而导致缓存被误删或命中异常。

|项|cacheId|cacheName|
|---|---|---|
|作用域|全局|单条缓存规则|
|影响|所有 Workbox cache|单个 runtimeCaching|
|使用场景|应用级隔离|业务级区分|
###### skipWaiting：
决定 **“新版 Service Worker 下载完成后，是否立即上位接管页面”**。
`skipWaiting` 是为了打破更新机制中的“等待”僵局。
`skipWaiting` 是新旧逻辑更替的“缓冲带”，设为 `false` 是为了安全，通过手动触发它是为了体验。
设置为**true：
- **作用**：一旦新版 SW 安装完成，它会立即强制杀掉旧版 SW，接管当前页面。
- **优点**：更新速度最快，用户下次刷新（甚至不刷新）就能用上新逻辑。
- **风险**：**非常危险！** 如果你的新版 SW 修改了拦截逻辑或删除了某些缓存，而用户还在操作旧页面，可能会导致页面请求报错、白屏或逻辑崩溃。
设置为**false：
- **作用**：新版 SW 安装后会乖乖待在后台（Waiting 状态），直到旧版 SW 随页面关闭而退出。
- **优点**：最安全。保证了“旧代码配旧 SW，新代码配新 SW”，不会起冲突。
- **缺点**：更新缓慢。如果用户不关浏览器，可能永远用不到新版逻辑。
###### clientsClaim：
默认情况下，即使 Service Worker (SW) 已经激活成功，它也**不会**拦截当前已经打开的页面。
如果skipWaiting设置为true，并且clientsClaim设置为true，让 Service Worker 在激活（Activate）的第一时间，立即强行接管（Claim）当前已经打开的所有页面。
**开启 `clientsClaim: true` 后**：
1. 用户第一次访问页面 -> SW 注册并安装。
2. SW 一旦激活，立即向浏览器宣告：“我现在要管辖所有已经打开的页面！”
3. **当前页面剩下的 API 请求立即开始被 SW 拦截和缓存。**

`skipWaiting` + `clientsClaim`：黄金组合
这两个属性通常是绑在一起使用的。它们分工明确：
- **`skipWaiting: true`**：解决的是**“新旧 SW 权力交接”**的问题。让新 SW 别排队，赶紧上位。
- **`clientsClaim: true`**：解决的是**“新 SW 上位后立刻干活”**的问题。让它别等下次刷新，现在就开始拦截。
###### handler：
- NetworkFirst ：优先从网络获取。如果网络请求成功，就返回结果并更新缓存；如果断网或请求失败，则回退到缓存中拿旧数据，适用实时性高的场景。
- CacheFirst：优先从缓存获取。如果缓存里有，直接返回，不再发网络请求；如果缓存没有，才去请求网络并存入缓存。适用极少更新的静态资源。
- StaleWhileRevalidate：过时即重新验证，最推荐的 API 优化方案。立即从缓存返回数据（秒开），同时在后台默默发起网络请求更新缓存。适用场景： 追求**极致加载速度**，且数据更新不那么频繁的场景。**注意：** 用户本次看到的是上次的数据，下次打开才是最新的。
- NetworkOnly：仅限网络，强制只从网络获取，完全不查找缓存，也不存储缓存。
- CacheOnly：仅限缓存，强制只从缓存获取。如果缓存没有，请求就会失败。极少使用。
###### maxEntries VS maxAgeSeconds：
maxEntries控制“数量上限”，控的是“最多能存多少”，不是“一定要存多少”。当缓存条目数 > maxEntries 时，开始淘汰旧数据，如果拦截的url是固定的，没有query参数或者query参数也是固定的，那么它的值可以写1，但是如果像是查新参数会变，或者是分页等情况，可以将它的值设置为比较大的数，用以缓存更多数据。
在固定接口场景下：
- **`maxAgeSeconds` 才是主控**
- `maxEntries` 基本只是安全兜底

#### 更新机制：
- **安装 (Installing)**：浏览器发现服务器上的 `service-worker.js` 变了，下载并安装新版本。
- **等待 (Waiting)**：新版本安装好了，但它**不能立即生效**。因为此时旧版本正在控制着当前页面，如果新版本突然接管，可能会导致页面逻辑冲突（比如旧页面去请求新 SW 里已经不存在的资源）。
- **激活 (Activated)**：只有当用户**关闭所有标签页**并重新打开，旧 SW 彻底死亡后，新 SW 才会上位。

```js
// webpack.config.js
const { GenerateSW } = require('workbox-webpack-plugin');
module.exports = {
	plugins: [
		new GenerateSW({
			cacheId: 'my-api-cache-v1', 
			
			skipWaiting: true, 
			clientsClaim: true,
			
			// 不缓存任何 Webpack 生成的 JS/CSS
			// 空 precache manifest
			// 完全禁用 precache（预缓存），生成的 Service Worker 不会预缓存任何构建产物，只保留 runtimeCaching 等运行时逻辑。
			chunks: [], 
			exclude: [/.*/],
			
			runtimeCaching: [
				{
					urlPattern: /^https:\/\/jsonplaceholder\.typicode\.com\/posts/, 
					handler: 'StaleWhileRevalidate',
					options: {
						cacheName: 'api-posts-cache',
						expiration: { 
							maxEntries: 1, // 拦截一个固定的url，设置为1就行，如果匹配的url有参数，那么就设置为一个比较大的数
							maxAgeSeconds: 3600 // 缓存1小时
						},
						// 启用 broadcastUpdate 插件 
						plugins: [ 
							new workbox.broadcastUpdate.BroadcastUpdatePlugin({ 
								channelName: 'api-updates', 
							}), 
						],
					}
				}
			]
		})
	]
}
```

项目入口文件 src/index.js
```js
import React from 'react'; 
import { createRoot } from 'react-dom/client'; 
import App from './App'; 
import { register } from './registerSW'; 

createRoot(document.getElementById('root')).render(<App />);

// 仅生产环境注册 SW 
if (process.env.NODE_ENV === 'production') { 
	register(); 
}
```

项目中新建 `src/registerSW.js`
```js
export function register() {
  if (!('serviceWorker' in navigator)) return;
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register(
        '/service-worker.js'
      );
      listenForUpdates(registration);
      listenForMessages();
    } catch (e) {
      console.error('SW registration failed:', e);
    }
  });
}
```

```js
function listenForUpdates(registration: ServiceWorkerRegistration) {
  if (!registration) return;
  registration.addEventListener('updatefound', () => {
    const newWorker = registration.installing;
    if (!newWorker) return;
    newWorker.addEventListener('statechange', () => {
      if (
        newWorker.state === 'installed' &&
        navigator.serviceWorker.controller // 首次安装也会触发 installed，所有要判断controller
      ) {
        // 说明有新版本 SW
        notifyUpdateAvailable();
      }
    });
  });
}
```

```js
function listenForMessages() {
  navigator.serviceWorker.addEventListener('message', event => {
    if (event.data?.type === 'API_DATA_UPDATED') {
      notifyDataUpdated();
    }
  });
}
```

```js
function notifyUpdateAvailable() {
  const confirmed = window.confirm(
    '发现新版本，是否立即刷新？'
  );
  if (confirmed) {
    window.location.reload();
  }
}

function notifyDataUpdated() {
  const confirmed = window.confirm(
    '数据已更新，是否刷新页面获取最新内容？'
  );
  if (confirmed) {
    window.location.reload();
  }
}
```

整个事件流：
用户打开页面
↓
SW 注册
↓
新版本发布
↓
浏览器发现新 SW
↓
updatefound
↓
installed
↓
页面提示
↓
用户刷新
↓
新 SW 生效

用Nginx部署的一些问题：
- SW 是 **静态文件**
- 路径决定 **控制范围（scope）** 
- Nginx 必须 **禁止强缓存**
- MIME 类型必须正确

/var/www/html/ (dist 目录)
├── index.html
├── service-worker.js  <-- 就在这里，根目录下，也就是：https://your-domain.com/service-worker.js
├── static/
│   ├── js/
│   └── css/

```Nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/html;

    # 1. 核心：禁止 service-worker.js 缓存
    # 确保浏览器每次打开网页都会去服务器对比这个文件是否有更新
    location = /service-worker.js {
        add_header Cache-Control "no-store, no-cache, must-revalidate, proxy-revalidate, max-age=0";
        expires off;
        proxy_no_cache 1;
        default_type application/javascript; # 明确 `Content-Type`
    }

    # 2. 你的业务 HTML 建议也设置协商缓存
    location / {
        try_files $uri $uri/ /index.html;
        add_header Cache-Control "no-cache"; # 协商缓存，每次都去服务器问一下 Etag
    }

    # 3. 静态资源（JS/CSS）可以设置长久强缓存
    # 因为 Webpack 给它们加了 contenthash，文件名变了自然会下载新的
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 一个部署时的“巨坑”

如果你使用了 **CDN**，并且把所有的静态资源（JS/CSS）都推到了 `cdn.example.com`，而你的网页在 `www.example.com`：
- **千万不要**把 `service-worker.js` 放在 CDN 域名下注册。
- **必须**把 `service-worker.js` 放在你**主域名的服务器**上（和 `index.html` 在一起）。
- **原因**：Service Worker 默认不能跨域工作。如果它在 CDN 上，它就无法拦截主域名的 API 请求。