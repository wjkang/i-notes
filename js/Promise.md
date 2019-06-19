```js
const isFunction = variable => typeof variable === 'function'
const PENDING = "PENDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";
function XPromise(handle){
  this._status = PENDING;
  this._value = undefined;
  this._fulfilledQueues = [];
  this._rejectedQueues = [];
  try {
    handle(this._resolve.bind(this), this._reject.bind(this));
  } catch (err) {
    this._reject(err);
  }
}
XPromise.prototype._resolve = function(val) {
  let fun = ()=> {
      if (this._status !== PENDING) return
      // 依次执行成功队列中的函数，并清空队列
      const runFulfilled = (value) => {
        let cb;
        while (cb = this._fulfilledQueues.shift()) {
          cb(value)
        }
      }
      // 依次执行失败队列中的函数，并清空队列
      const runRejected = (error) => {
        let cb;
        while (cb = this._rejectedQueues.shift()) {
          cb(error)
        }
      }
      /* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
        当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
      */
      if (val instanceof XPromise) {
        val.then(value => {
          this._value = value
          this._status = FULFILLED
          runFulfilled(value)
        }, err => {
          this._value = err
          this._status = REJECTED
          runRejected(err)
        })
      } else {
        this._value = val
        this._status = FULFILLED
        runFulfilled(val)
      }
    }
    // 为了支持同步的Promise，这里采用异步调用
    setTimeout(fun, 0)
};
XPromise.prototype._reject = function(err) {
  if (this._status !== PENDING) return;
  // 依次执行失败队列中的函数，并清空队列
  let fun = ()=>{
    this._status = REJECTED
    this._value = err
    let cb;
    while (cb = this._rejectedQueues.shift()) {
      cb(err)
    }
  }
  // 为了支持同步的Promise，这里采用异步调用
  setTimeout(fun, 0)
};
XPromise.prototype.then = function(onFulfilled, onRejected) {
  let _this=this;
  // 返回一个新的Promise对象
  return new XPromise((onFulfilledNext, onRejectedNext) => {
    // 封装一个成功时执行的函数
    let fulfilled=function(value){
      try{
        if (!isFunction(onFulfilled)) {
            onFulfilledNext(value)
          } else {
            let res = onFulfilled(value);
            if (res instanceof XPromise) {
              // 如果当前回调函数返回XPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext)
            } else {
              //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res)
            }
          }
      }catch(err){
        // 如果函数执行出错，新的Promise对象的状态为失败
        onRejectedNext(err)
      }
    }
    // 封装一个失败时执行的函数
    let rejected = function(error){
      try {
        if (!isFunction(onRejected)) {
          onRejectedNext(error)
        } else {
            let res = onRejected(error);
            if (res instanceof XPromise) {
              // 如果当前回调函数返回XPromise对象，必须等待其状态改变后在执行下一个回调
              res.then(onFulfilledNext, onRejectedNext)
            } else {
              //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
              onFulfilledNext(res)
            }
        }
      } catch (err) {
        // 如果函数执行出错，新的Promise对象的状态为失败
        onRejectedNext(err)
      }
  }
    switch (_this._status) {
    // 当状态为pending时，将then方法回调函数加入执行队列等待执行
    case PENDING:
      _this._fulfilledQueues.push(onFulfilled);
      _this._rejectedQueues.push(onRejected);
      break;
    // 当状态已经改变时，立即执行对应的回调函数
    case FULFILLED:
      fulfilled(_this._value);
      break;
    case REJECTED:
      rejected(_this._value);
      break;
  }
  })
}
XPromise.prototype.catch=function(onRejected) {
  return this.then(undefined, onRejected)
}
XPromise.prototype.finally=function(cb) {
  return this.then(
    value => XPromise.resolve(cb()).then(() => value),
    reason => XPromise.resolve(cb()).then(() => { throw reason })
  );
}
XPromise.resolve=function(value) {
  // 如果参数是XPromise实例，直接返回这个实例
  if (value instanceof XPromise) return value
  return new XPromise(resolve => resolve(value))
}
XPromise.reject=function(value) {
  return new XPromise((resolve ,reject) => reject(value))
}
XPromise.all=function(list) {
  return new XPromise((resolve, reject) => {
    let values = []
    let count = 0
    for (let [i, p] of list.entries()) {
      // 数组参数如果不是XPromise实例，先调用XPromise.resolve
      this.resolve(p).then(res => {
        values[i] = res
        count++
        // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
        if (count === list.length) resolve(values)
      }, err => {
        // 有一个被rejected时返回的MyPromise状态就变成rejected
        reject(err)
      })
    }
  })
}
XPromise.race=function(list) {
  return new XPromise((resolve, reject) => {
    for (let p of list) {
      // 只要有一个实例率先改变状态，新的XPromise的状态就跟着改变
      this.resolve(p).then(res => {
        resolve(res)
      }, err => {
        reject(err)
      })
    }
  })
}


XPromise.resolve(1).then(()=>{console.log(1)})
console.log(2);

```

**简易版**
```js
// 简易版本的promise 
// 第一步： 列出三大块  this.then   resolve/reject   fn(resolve,reject)
// 第二步： this.then负责注册所有的函数   resolve/reject负责执行所有的函数 
// 第三步： 在resolve/reject里面要加上setTimeout  防止还没进行then注册 就直接执行resolve了
// 第四步： resolve/reject里面要返回this  这样就可以链式调用了
// 第五步： 三个状态的管理 pending fulfilled rejected

// *****promise的链式调用 在then里面return一个promise 这样才能then里面加上异步函数
// 加上了catch
function PromiseM(fn) {
    var value = null;
    var callbacks = [];
    //加入状态 为了解决在Promise异步操作成功之后调用的then注册的回调不会执行的问题
    var state = 'pending';
    var _this = this;

    //注册所有的回调函数
    this.then = function (fulfilled, rejected) {
        //如果想链式promise 那就要在这边return一个new Promise
        return new PromiseM(function (resolv, rejec) {
            //异常处理
            try {
                if (state == 'pending') {
                    callbacks.push(fulfilled);
                    //实现链式调用
                    return;
                }
                if (state == 'fulfilled') {
                    var data = fulfilled(value);
                    //为了能让两个promise连接起来
                    resolv(data);
                    return;
                }
                if (state == 'rejected') {
                    var data = rejected(value);
                    //为了能让两个promise连接起来
                    resolv(data);
                    return;
                }
            } catch (e) {
                _this.catch(e);
            }
        });
    }

    //执行所有的回调函数
    function resolve(valueNew) {
        value = valueNew;
        state = 'fulfilled';
        execute();
    }

    //执行所有的回调函数
    function reject(valueNew) {
        value = valueNew;
        state = 'rejected';
        execute();
    }

    function execute() {
        //加入延时机制 防止promise里面有同步函数 导致resolve先执行 then还没注册上函数
        setTimeout(function () {
            callbacks.forEach(function (cb) {
                value = cb(value);
            });
        }, 0);
    }

    this.catch = function (e) {
        console.log(JSON.stringify(e));
    }

    //经典 实现异步回调
    fn(resolve, reject);
}

```