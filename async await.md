无论 `await` 后面是什么（普通值、Promise、thenable对象），都会先用 `Promise.resolve()` 包装这个值，然后把 `await` 后面的代码**作为这个 Promise 的 `.then()` 回调**，  这个回调**会被放入微任务队列**。然后跳出async1函数，执行其他代码，当遇到promise函数的时候，会注册promise.then()函数到微任务队列，注意此时微任务队列里面已经存在await后面的微任务。所以这种情况会先执行await后面的代码（async1 end），再执行async1函数后面注册的微任务代码(promise1,promise2)。
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
// 执行结果中没有async1 success 和 async1 end
// 因为promise没有执行resolve，导致promise一直处在pending态，await一直等待，就不会有微任务加入到队列中，async1().then(...) 中的回调不会执行

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