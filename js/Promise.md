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