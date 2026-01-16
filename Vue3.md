## 编译阶段性能优化：
- diff算法优化：静态标记
```js
export const enum PatchFlags {
  TEXT = 1,// 1 动态的文本节点
  CLASS = 1 << 1,  // 2 动态的 class
  STYLE = 1 << 2,  // 4 动态的 style
  PROPS = 1 << 3,  // 8 动态属性，不包括类名和样式
  FULL_PROPS = 1 << 4,  // 16 动态 key，当 key 变化时需要完整的 diff 算法做比较
  HYDRATE_EVENTS = 1 << 5,  // 32 表示带有事件监听器的节点
  STABLE_FRAGMENT = 1 << 6,   // 64 一个不会改变子节点顺序的 Fragment
  KEYED_FRAGMENT = 1 << 7, // 128 带有 key 属性的 Fragment
  UNKEYED_FRAGMENT = 1 << 8, // 256 子节点没有 key 的 Fragment
  NEED_PATCH = 1 << 9,   // 512  表示只需要non-props修补的元素 (non-props不知道怎么翻才恰当~)
  DYNAMIC_SLOTS = 1 << 10,  // 1024 动态的solt
  DEV_ROOT_FRAGMENT = 1 << 11, //2048 表示仅因为用户在模板的根级别放置注释而创建的片段。 这是一个仅用于开发的标志，因为注释在生产中被剥离。
 
  //以下两个是特殊标记
  HOISTED = -1,  // 表示已提升的静态vnode,更新时调过整个子树
  BAIL = -2 // 指示差异算法应该退出优化模式
```
- 静态提升
不参与更新的元素，会做静态提升，只会被创建一次，在渲染时直接复用。
免去了重复的创建操作，优化内存。特殊标志是`负整数`表示永远不会用于 `Diff`。
```js
// 没有静态提升：
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock(_Fragment, null, [
    _createVNode("span", null, "你好"),
    _createVNode("div", null, _toDisplayString(_ctx.message), 1 /* TEXT */)
  ], 64 /* STABLE_FRAGMENT */))
}
```

```js
// 有静态提升：
const _hoisted_1 = /*__PURE__*/_createVNode("span", null, "你好", -1 /* HOISTED */)

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock(_Fragment, null, [
    _hoisted_1,
    _createVNode("div", null, _toDisplayString(_ctx.message), 1 /* TEXT */)
  ], 64 /* STABLE_FRAGMENT */))
}
```
- 事件监听缓存。默认情况下绑定事件行为会被视为动态绑定，所以每次都会去追踪它的变化
```js
// 没开启事件监听器缓存：
export const render = /*__PURE__*/_withId(function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("button", { onClick: _ctx.onClick }, "点我", 8 /* PROPS */, ["onClick"])
                                             // PROPS=1<<3,// 8 //动态属性，但不包含类名和样式
  ]))
})
```

```js
// 开启事件侦听器缓存后：
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("button", {
      onClick: _cache[1] || (_cache[1] = (...args) => (_ctx.onClick(...args)))
    }, "点我")
  ]))
}
```
- SSR优化
当静态内容大到一定量级时候，会用createStaticVNode方法在客户端去生成一个static node，这些静态node，会被直接innerhtml，就不需要创建对象，然后根据对象渲染。
## 依赖收集
```js
const targetMap = new WeakMap();
let activityEffect = null;
function track(target, key) {
	if (!activityEffect) return;
	let depsMap = targetMap.get(target);
	if (!depsMap) {
		depsMap = new Map();
		targetMap.set(target, depsMap);
	}
	let dep = depsMap.get(key);
	if (!dep) {
		dep = new Set();
		depsMap.set(key, dep);
	}
	dep.add(activityEffect);
}
function trigger(target, key) {
	const depsMap = targetMap.get(target);
	if (!depsMap) return;
	const dep = depsMap.get(key);
	if (!dep) return;
	dep.forEach(effect => effect())
}

targetMap = WeakMap(target, Map(key, Set(effect)))
```

```js
function reactive(target) {
	return new Proxy(target, {
		get(target, key, receiver) {
			const result = Reflect.get(target, key, receiver);
			track(target, key);
			return result;
		},
		set(target, key, value, receiver) {
			const result = Reflect.set(target, key, value, receiver);
			trigger(target, key);
			return result;
		}
	})
}
```
![截屏2023-05-14 10.41.12.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/036d44b6edb54b25902988dc5723685e~tplv-k3u1fbpfcp-watermark.image?)

![截屏2023-05-14 10.49.09.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb9bd7481f9648a89ae7780d0bbd9a3a~tplv-k3u1fbpfcp-watermark.image?)