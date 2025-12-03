1. 原型链继承
- 将子类的原型指向父类的实例
- 缺点：子类的实例访问子类的属性，若没有，则通过原型链访问父类上的属性，如果修改父类上属性的值，则其他实例也会被影响，因为不同实例都指向同一个原型对象，内存共享。

```js
function Father() {}
function Child() {}
Child.prototype = new Father();
```
2. 构造函数继承
- 通过call, apply实现继承
- 缺点：
    - 每次实例化，父类都会执行一遍call
    - 实例访问不到父类原型上的属性和方法
```js
function Father() {}
function Child() {
    Father.call(this, args);
}
```
3. 组合式继承
- 将原型链继承和构造函数继承结合起来
- 缺点：两种继承方式的缺点都没法避免

```js
function Father() {}
function Child() {
    Father.call(this, args);
}
Child.prototype = new Father();
Child.prototype.constructor = Child;
```
4. 原型式继承
- 基于已有的对象创建新对象
- 缺点：引用类型共享，无法实现多继承。

```js
function Father() {};
let Child = Object.create(Father);
```
5. 寄生式继承
- 利用Object.create获得一份目标对象的浅拷贝，解决原型上的属性方法被覆盖的问题。
- 缺点：与原型式继承一样，无法实现多继承

```js
Child.prototype = new Father();
Child.prototype.constructor = Child;
替换为：
Child.prototype = Object.create(Father.prototype);
Child.prototype.constructor = Child;
```
6. 寄生组合式继承

```js
function Father() {};
function Child() {
    Father.call(this, args);
}
Child.prototype = Object.create(Father.prototype);
Child.prototype.constructor = Child;
```

![1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6a433ff87cd47df83ffbeb7c1b30bc9~tplv-k3u1fbpfcp-watermark.image?)