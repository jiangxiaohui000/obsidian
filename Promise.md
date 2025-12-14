```js
Promise.resolve()
  .then(() => {
    return new Error('error!!!')
  })
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })
```
解析：  
.then 或者 .catch 中 return 一个 error 对象并不会抛出错误，所以不会被后续的 .catch 捕获，需要改成其中一种：

```js
return Promise.reject(new Error('error!!!'))
throw new Error('error!!!')
```
因为返回任意一个非 promise 的值都会被包裹成 promise 对象，即 return new Error('error!!!') 等价于 return Promise.resolve(new Error('error!!!'))。

---
```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
  // 1
```
解析：  
.then 或者 .catch 的参数期望是函数，非函数 onFulfilled → 采用默认回调：`value => value`，整体等价：
```js
Promise.resolve(1)
  .then(value => value)   // 传递 1
  .then(value => value)   // 仍然是 1
  .then(console.log)      // 打印 1
```
---
```js
async function getResult() {
    await new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(1);
            console.log(1);
        }, 1000);
    })
    await new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(2);
            console.log(2);
        }, 500);
    })
    await new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(3);
            console.log(3);
        }, 100);
    })

}
getResult()；// 1 2 3
// await需要等待上一个promise resolve之后才会执行
// 如果把上面代码中resolve去掉，那么后面的await就不会执行了。
```

---

```js
function red() {
  console.log('red');
}

function green() {
  console.log('green');
}

function yellow() {
  console.log('yellow');
}


let myLight = (timer, cb) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      cb();
      resolve();
    }, timer);
  });
};


let myStep = () => {
  Promise.resolve()
	  .then(() => myLight(3000, red))
	  .then(() => myLight(2000, green))
	  .then(() => myLight(1000, yellow))
	  .then(() => myStep())
};
myStep();
```
---
`Promise.resolve(v)`不等于`new Promise(resolve => resolve(v))`

```js
// 首先，v是一个实例化的promise，且状态为fulfilled
let v = new Promise(resolve => {
  console.log("begin");
  resolve("then");
});

// 然后，在promise里面resolve一个状态为fulfilled的promise
// 以下分两种情况：

// 模式一 new Promise里的resolve()
new Promise(resolve => {
  resolve(v);
}).then((v)=>{
  console.log(v)
});
// begin->1->2->3->then->4 可以发现then推迟了两个时序
// 推迟原因：
// 浏览器会创建一个 PromiseResolveThenableJob 去处理这个 Promise 实例，这是一个微任务。
// 等到下次循环到来这个微任务会执行，也就是PromiseResolveThenableJob执行中的时候，
// 因为这个Promise 实例是fulfilled状态，所以又会注册一个它的.then()回调
// 又等一次循环到这个Promise 实例它的.then()回调执行后，才会注册下面的这个.then()，
// 于是就被推迟了两个时序

//  模式二 Promise.resolve(v)直接创建
Promise.resolve(v).then((v)=>{
  console.log(v)
});
// begin->1->then->2->3->4 可以发现then的执行时间正常了，第一个执行的微任务就是下面这个.then
// 原因：Promise.resolve()如果参数是promise会直接返回这个promise实例，不会做任何处理

new Promise(resolve => {
  console.log(1);
  resolve();
})
  .then(() => {
    console.log(2);
  })
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(4);
  });
```

手写promise的简单实现
```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class myPromise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFullfilledCallback = [];
    this.onRejectedCallback = [];
    const resolve = (value) => {
      if(this.status === PENDING) {
        this.status = FULFILLED;
        this.value = value;
        this.onFullfilledCallback.forEach(item => item(value));
      }
    }
    const reject = (reason) => {
      if(this.status === PENDING) {
        this.status = REJECTED;
        this.reason = reason;
        this.onRejectedCallback.forEach(item => item(reason));
      }
    }
    try {
      executor(resolve, reject);
    } catch(e) {
      reject(e);
    }
  }
  static resolve(value) {
    if(value instanceof myPromise) {
      return value;
    } else {
      return new myPromise(resolve => resolve(value))
    }
  }
  static reject(reason) {
      return new myPromise(reject => reject(reason))
  }
  then(onFullfilled, onRejected) {
    onFullfilled = typeof onFullfilled === 'function' ? onFullfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason};
    const promise2 = new myPromise((resolve, reject) => {
      const fullfilledMicrotask = () => {
        try {
          queueMicrotask(() => {
            const x = onFullfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          })
        } catch(e) {
          reject(e);
        }
      }
      const rejectedMicrotask = () => {
        try {
          queueMicrotask(() => {
            const x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          })
        } catch(e) {
          reject(e);
        }
      }
      if(this.status === FULFILLED) {
        fullfilledMicrotask();
      } else if(this.status === REJECTED) {
        rejectedMicrotask();
      } else if(this.status === PENDING) {
        this.onFullfilledCallback.push(fullfilledMicrotask);
        this.onRejectedCallback.push(rejectedMicrotask);
      }
    });
    return promise2;
  }
}
```

```js
Promise.prototype.all = function(arr) {
  return new Promise((resolve, reject) => {
    let count = 0;
    let result = [];
    let len = arr.length;
    if(len === 0) {
      return resolve(result);
    }
    for(let i = 0; i < len; i++) {
      Promise.resolve(arr[i])
	    .then(res => {
          result[i] = res;
          count++;
          if (count === len) {
            resolve(result);
          }
        })
        .catch(reason => reject(reason));
    }
  })
}
```

```js
Promise.prototype.race = function(arr) {
  return new Promise((resolve, reject) => {
    if (typeof arr[Symbol.iterator] !== 'function') {
      return reject(new Error(''对象不可迭代''));
    }
    if (arr.length === 0) {
      return; // 数据为空，Promise 永远 pending
    }
    for (let item of arr) {
	  Promise.resolve(item).then(resolve, reject);
    }
  })
}
```

```js
Promise.prototype.allSettled = function(arr) {
  return new Promise(resolve => {
    let len = arr.length;
    let result = [];
    let count = 0;
    if(len === 0) {
      return resolve(result);
    }
    for(let i = 0; i < len; i++) {
      Promise.resolve(arr[i])
      .then(res => {
        count++;
        result[i] = {
          status: 'fulfilled',
          value: res
        }
        if(count === len) {
          resolve(result);
        }
      })
      .catch(reason => {
        count++;
        result[i] = {
          status: 'rejected',
          reason: reason
        }
        if(count === len) {
          resolve(result);
        }
      })
    }
  })
}
```

```js
Promise.prototype.finally = function(fn) {
  return this.then(
    value => Promise.resolve(fn()).then(() => value),
    reason => Promise.resolve(fn()).then(() => Promise.reject(reason))
  );
}
```

```js
Promise.prototype.any = (arr) => {
	return new Promise((resolve, reject) => {
		let errors = [];
		let count = 0;
		let len = arr.length;
		if (len === 0) {
			return reject([]);
		}
		for (let i = 0; i < len; i++) {
			Promise.resolve(arr[i])
			.then(res => {
				resolve(res);
			})
			.catch(reason => {
				count++;
				errors[i] = reason;
				if (count === len) {
					reject(errors);
				}
			})
		}
	})
}
```

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