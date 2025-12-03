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
.then 或者 .catch 的参数期望是函数，传入非函数则会发生**值穿透**。原理上是当then中传入的不算函数，则这个`promise`返回上一个`promise`的值，这就是发生值穿透的原因.

---
.then 的第二个处理错误的函数捕获不了第一个处理成功的函数抛出的错误，而后续的 .catch 可以捕获之前的错误


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
  Promise.resolve().then(() => {
    return myLight(3000, red);
  }).then(() => {
    return myLight(2000, green);
  }).then(()=>{
    return myLight(1000, yellow);
  }).then(()=>{
    myStep();
  })
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
// 原因：Promise.resolve()API如果参数是promise会直接返回这个promise实例，不会做任何处理


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
    this.value = '';
    this.reason = '';
    this.onFullfilledCallback = [];
    this.onRejectedCallback = [];
    function resolve(value) {
      if(this.status === PENDING) {
        this.status = FULFILLED;
        this.value = value;
        this.onFullfilledCallback.forEach(item => item(value));
      }
    }
    function reject(reason) {
      if(this.status === PENDING) {
        this.status = REJECTED;
        this.reason = reason;
        this.onRejectedCallback.forEach(item => item(reason));
      }
    }
    try {
      executor(resolve, reject);
    } catch(e) {
      this.reject(e);
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
      Promise.resolve(arr[i]).then(res => {
        result[i] = res;
        count++;
        if(count === len) {
          resolve(result);
        }
      }, reason => {
        reject(reason);
      });
    }
  })
}
```


```js
Promise.prototype.race = function(arr) {
  return new Promise((resolve, reject) => {
    if(arr.length === 0) {
      return resolve()
    }
    arr.forEach(item => {
      Promise.resolve(item).then(res => {
        resolve(res);
      }, reason => {
        reject(reason);
      })
    })
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
      Promise.resolve(arr[i]).then(res => {
        count++;
        result[i] = {
          status: 'fulfilled',
          value: res
        }
        if(count === len) {
          resolve(result);
        }
      }, reason => {
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
  //
  return this.then(
    value => Promise.resolve(fn()).then(() => value),
    reason => Promise.resolve(fn()).then(() => { throw reason })
  )
  //
  return this.then(val => {
    return Promise.resolve(fn()).then(() => {
      return val;
    });
  }, reason => {
    return Promise.resolve(fn()).then(() => {
      throw reason;
    });
  });
}
```