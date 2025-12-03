- 只有声明被提升，初始化不会被提升
- 声明会被提升到当前作用域的顶端

**函数提升**

- 函数声明

```js
console.log(square(5)); // 25
function square(n) {
  return n * n;
}
```
编译后

```js
function square (n) {
  return n * n;
}
console.log(square(5)); // 25
```
- 函数表达式

```js
console.log(square); // undefined
console.log(square(5)); // square is not a function =》 初始化并未提升，此时 square 值为 undefined
var square = function (n) { 
  return n * n; 
}
```
编译后

```js
var square
console.log(square); // undefined =》赋值没有被提升
console.log(square(5)); // square is not a function =》 square 值为 undefined 故报错
square = function (n) { 
  return n * n; 
}
```


```js
var tmp = new Date();
function fn(){
  console.log(tmp);
  if(false){
    var tmp = 'hello nanjiu';
  }
}
fn(); // undefined
// 为什么是undefined，因为变量提升是发生在代码编译阶段，在执行之前，所以编译完是这样的
function fn(){
    var tmp; // 声明被提升
    console.log(tmp); // undefined，因为初始化并没有被提升
    if (false) {
      tmp = 'hello nanjiu'; // 这里的赋值部分不会被提升
    }
}
```
## 优先级

-   **函数提升在变量提升之前**

1.  变量的问题，莫过于声明和赋值两个步骤，而这两个步骤是分开的。
2.  函数声明被提升时，声明和赋值两个步骤都会被提升，而普通变量却只能提升声明步骤，而不能提升赋值步骤。
3.  变量被提升过后，先对提升上来的所有对象统一执行一遍声明步骤，然后再对变量执行一次赋值步骤。而执行赋值步骤时，会**优先执行函数变量的赋值步骤，再执行普通变量的赋值步骤**。


私有作用域(函数作用域)，带 `var` 的是私有变量。不带 `var` 的是会向上级作用域查找，如果上级作用域也没有那么就一直找到 window 为止，这个查找过程叫`作用域链`。

在 `var 和 function` 同名的变量提升的条件下，函数会先执行。所以输出的结果都是一样的。换一句话说，`var 和 function` 的变量同名 `var` 会先进行变量提升，但是在变量提升阶段，函数声明的变量会覆盖 `var` 的变量提升，所以直接结果总是函数先执行优先。

```js
console.log(fn);
var fn = 2019;
console.log(fn);
function fn(){}

/* 输出
    fn(){}
    2019
/
```

思考题（字节）
```js
let a = 0, b = 0;
function fn(a) {
  fn = function fn2(b) {
    console.log(a, b)
    console.log(++a+b)
  }
  console.log('a', a++)
}
fn(1);
fn(2);
```
思考题分析：
- fn(1)执行时，参数1传进了函数fn，进入到fn函数内，首先fn被赋值为fn2，此时fn2中的a被赋值了1，

![1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c01602551fcd496da3464676ecc72155~tplv-k3u1fbpfcp-watermark.image?)
- 然后console打印，所以输出 'a' 1，但因为a++的缘故，在打印后变为了2。

![2.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cb92782c712494ca5c277e8906c0ada~tplv-k3u1fbpfcp-watermark.image?)
- 然后fn(2)执行，此时fn函数已经是fn2了，所以两行console打印，上一步a已经是2，此时传进来的参数2赋值给了b，第一行打印为2，2。

![3.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86b3d8b0e87d4c1bb074f600c0160989~tplv-k3u1fbpfcp-watermark.image?)
- 第二行打印就好理解了，++a是3，再加2，就是5了。

[前端面试必考—JavaScript变量提升和函数提升详解](https://segmentfault.com/a/1190000038344251)

[彻底解决 JS 变量提升| 一题一图，超详细包教包会](https://juejin.cn/post/6933377315573497864)
