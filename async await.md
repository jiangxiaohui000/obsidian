如果await 后面直接跟的为一个变量，比如：await 1；这种情况的话相当于直接把await后面的代码注册为一个微任务，可以简单理解为promise.then(await下面的代码)。然后跳出async1函数，执行其他代码，当遇到promise函数的时候，会注册promise.then()函数到微任务队列，注意此时微任务队列里面已经存在await后面的微任务。所以这种情况会先执行await后面的代码（async1 end），再执行async1函数后面注册的微任务代码(promise1,promise2)。
```js
console.log('script start')

async function async1() {
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2 end')
}
async1()

setTimeout(function() {
    console.log('setTimeout')
}, 0)

new Promise(resolve => {
    console.log('Promise')
    resolve()
})
.then(function() {
    console.log('promise1')
})
.then(function() {
    console.log('promise2')
})

console.log('script end')

// script start
// async2 end
// Promise
// script end
// async1 end
// promise1
// promise2
// setTimeout
```
如果await后面跟的是一个异步函数的调用

```js
console.log('script start')

async function async1() {
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2 end')
    return Promise.resolve().then(()=>{
      console.log('async2 end1')
    })
}
async1()

setTimeout(function() {
    console.log('setTimeout')
}, 0)

new Promise(resolve => {
    console.log('Promise')
    resolve()
})
.then(function() {
    console.log('promise1')
})
.then(function() {
    console.log('promise2')
})

console.log('script end')

// script start
// async2 end
// Promise
// script end
// async2 end1
// promise1
// promise2
// async1 end
// setTimeout
```
此时执行完await并不先把await后面的代码注册到微任务队列中去，而是执行完await之后，直接跳出async1函数，执行其他代码。然后遇到promise的时候，把promise.then注册为微任务。其他代码执行完毕后，需要回到async1函数去执行剩下的代码，然后把await后面的代码注册到微任务队列当中，注意此时微任务队列中是有之前注册的微任务的。所以这种情况会先执行async1函数之外的微任务(promise1,promise2)，然后才执行async1内注册的微任务(async1 end).


```js
async function async1() {  
  console.log('async1 start')  
  await new Promise(resolve => {  
    console.log('promise1')  
  })  
  console.log('async1 success')  
  return 'async1 end'  
}  
console.log('srcipt start')  
async1().then(res => console.log(res))  
console.log('srcipt end')

// srcipt start
// async1 start
// promise1
// srcipt end
// 执行结果中没有console.log('async1 success') 和 return 'async1 end'，因为promise没有执行resolve。

// 如果改成这样：
async function async1() {  
  console.log('async1 start')  
  await new Promise(resolve => {  
    console.log('promise1')
    resolve() // 加上resolve
  })  
  console.log('async1 success')  
  return 'async1 end'  
}  
console.log('srcipt start')  
async1().then(res => console.log(res))  
console.log('srcipt end')

// srcipt start
// async1 start
// promise1
// srcipt end
// async1 success
// async1 end
```

[async/await 原理及执行顺序分析](https://juejin.cn/post/6844903988584775693)

[消灭异步回调，还得是async-await](https://juejin.cn/post/7122478595787718663)