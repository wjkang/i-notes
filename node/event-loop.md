>当Node.js启动时会初始化event loop, 每一个event loop都会包含按如下顺序六个循环阶段

![](../img/2019-05-28-00-02-27.png)

* **timers 阶段**: 这个阶段执行setTimeout(callback) and setInterval(callback)预定的callback;
* **I/O callbacks 阶段**: 执行除了 close事件的callbacks、被timers(定时器，setTimeout、setInterval等)设定的callbacks、setImmediate()设定的callbacks之外的callbacks;
* **idle, prepare 阶段**: 仅node内部使用;
* **poll 阶段**: 获取新的I/O事件, 适当的条件下node将阻塞在这里;
* **check 阶段**: 执行setImmediate() 设定的callbacks;
* **close callbacks 阶段**: 比如socket.on(‘close’, callback)的callback会在这个阶段执行.


每一个阶段都有一个装有callbacks的FIFO queue(队列)，当event loop运行到一个指定阶段时， node将执行该阶段的FIFO queue(队列)，当队列callback执行完或者执行callbacks数量超过该阶段的上限时， event loop会转入下一下阶段。

!>上面六个阶段都不包括 process.nextTick()

**poll阶段**

poll阶段是衔接整个event loop各个阶段比较重要的阶段。

在node.js里，任何异步方法（除timer,close,setImmediate之外）完成时，都会将其callback加到poll queue里,并立即执行。

poll 阶段有两个主要的功能：

* 处理poll队列（poll quenue）的事件(callback);
* 执行timers的callback,当到达timers指定的时间时;

如果event loop进入了 poll阶段，且代码未设定timer，将会发生下面情况：

* 如果poll queue不为空，event loop将同步的执行queue里的callback,直至queue为空，或执行的callback到达系统上限;

* 如果poll queue为空，将会发生下面情况：

  * 如果代码已经被setImmediate()设定了callback, event loop将结束poll阶段进入check阶段，并执行check阶段的queue (check阶段的queue是 setImmediate设定的)

  * 如果代码没有设定setImmediate(callback)，event loop将阻塞在该阶段等待callbacks加入poll queue;

如果event loop进入了 poll阶段，且代码设定了timer：

* 如果poll queue进入空状态时（即poll 阶段为空闲状态），event loop将检查timers,如果有1个或多个timers时间时间已经到达，event loop将按循环顺序进入 timers 阶段，并执行timer queue.

**例子1**

```js
var fs = require('fs');

function someAsyncOperation (callback) {
  // 花费2毫秒
  fs.readFile(__dirname + '/' + __filename, callback);
}

var timeoutScheduled = Date.now();
var fileReadTime = 0;

setTimeout(function () {
  var delay = Date.now() - timeoutScheduled;
  console.log('setTimeout: ' + (delay) + "ms have passed since I was scheduled");
  console.log('fileReaderTime',fileReadtime - timeoutScheduled);
}, 10);

someAsyncOperation(function () {
  fileReadtime = Date.now();
  while(Date.now() - fileReadtime < 20) {

  }
});
```

结果: 先执行someAsyncOperation的callback,再执行setTimeout callback

>-> node eventloop.js

>setTimeout: 22ms have passed since I was scheduled
fileReaderTime 2

解释：

当时程序启动时，event loop初始化：

1、timer阶段（无callback到达，setTimeout需要10毫秒）

2、i/o callback阶段，无异步i/o完成

3、忽略

4、poll阶段，阻塞在这里，当运行2ms时，fs.readFile完成，将其callback加入 poll队列，并执行callback， 其中callback要消耗20毫秒,等callback之行完，poll处于空闲状态，由于之前设定了timer，因此检查timers,发现timer设定时间是20ms，当前时间运行超过了该值，因此，立即循环回到timer阶段执行其callback,因此，虽然setTimeout的20毫秒，但实际是22毫秒后执行。

**例子2**

```js
var fs = require('fs');

function someAsyncOperation (callback) {
  var time = Date.now();
  // 花费9毫秒
  fs.readFile('/path/to/xxxx.pdf', callback);
}

var timeoutScheduled = Date.now();
var fileReadTime = 0;
var delay = 0;

setTimeout(function () {
  delay = Date.now() - timeoutScheduled;
}, 5);

someAsyncOperation(function () {
  fileReadtime = Date.now();
  while(Date.now() - fileReadtime < 20) {

  }
  console.log('setTimeout: ' + (delay) + "ms have passed since I was scheduled");
  console.log('fileReaderTime',fileReadtime - timeoutScheduled);
});
```

结果：setTimeout callback先执行，someAsyncOperation callback后执行

解释：

当时程序启动时，event loop初始化：

1、timer阶段（无callback到达，setTimeout需要10毫秒）

2、i/o callback阶段，无异步i/o完成

3、忽略

4、poll阶段，阻塞在这里，当运行5ms时，poll依然空闲，但已设定timer,且时间已到达，因此，event loop需要循环到timer阶段,执行setTimeout callback,由于从poll --> timer中间要经历check,close阶段,这些阶段也会消耗一定时间，因此执行setTimeout callback实际是7毫秒 然后又回到poll阶段等待异步i/o完成，在9毫秒时fs.readFile完成，其callback加入poll queue并执行。

**setTimeout 和 setImmediate**

二者非常相似，但是二者区别取决于他们什么时候被调用.

* setImmediate 设计在poll阶段完成时执行，即check阶段；
* setTimeout 设计在poll阶段为空闲时，且设定时间到达后执行；但其在timer阶段执行

其二者的调用顺序取决于当前event loop的上下文，如果他们在异步i／o callback之外调用，其执行先后顺序是不确定的

```js
setTimeout(function timeout () {
  console.log('timeout');
},0);

setImmediate(function immediate () {
  console.log('immediate');
});
```

但当二者在异步i/o callback内部调用时，总是先执行setImmediate，再执行setTimeout

```js
var fs = require('fs')

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout')
  }, 0)
  setImmediate(() => {
    console.log('immediate')
  })
})
```

>immediate
timeout

理解了event loop的各阶段顺序这个例子很好理解： 因为fs.readFile callback执行完后，程序设定了timer 和 setImmediate，因此poll阶段不会被阻塞进而进入check阶段先执行setImmediate，后进入timer阶段执行setTimeout

**process.nextTick()**

process.nextTick()不在event loop的任何阶段执行，而是在各个阶段切换的中间执行,即从一个阶段切换到下个阶段前执行。

```js
var fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('setTimeout');
  }, 0);
  setImmediate(() => {
    console.log('setImmediate');
    process.nextTick(()=>{
      console.log('nextTick3');
    })
  });
  process.nextTick(()=>{
    console.log('nextTick1');
  })
  process.nextTick(()=>{
    console.log('nextTick2');
  })
});
```

```
-> node eventloop.js
nextTick1
nextTick2
setImmediate
nextTick3
setTimeout
```

从poll —> check阶段，先执行process.nextTick，

nextTick1

nextTick2

然后进入check,setImmediate，

setImmediate

执行完setImmediate后，出check,进入close callback前，执行process.nextTick

nextTick3

最后进入timer执行setTimeout

setTimeout

process.nextTick()是node早期版本无setImmediate时的产物，node作者推荐我们尽量使用setImmediate。

当递归调用process.nextTick时，即使fs.readFile完成，其callback无机会执行：

```js
var fs = require('fs');
var starttime = Date.now();
var endtime = 0;

fs.readFile(__filename, () => {
  endtime = Date.now();
  console.log('finish reading time:',endtime - starttime);
});

var index = 0;

function nextTick () {
  if (index > 1000) return;
  index++;
  console.log('nextTick');
  process.nextTick(nextTick);
}

nextTick();
```
```
-> node eventloop.js
nextTick
nextTick
...
nextTick
nextTick
finish reading time: 246
```

将process.nextTick替换成setImmediate后，由于setImmediate只在check阶段执行，那么所有的callback都有机会执行：

```js
var fs = require('fs');

fs.readFile(__filename, () => {
  console.log('finish reading');
});

var index = 0;

function Immediate () {
  if (index > 100) return;
  index++;
  console.log('setImmediate');
  setImmediate(Immediate);
}

Immediate()
```

```
-> node eventloop.js
setImmediate
setImmediate
setImmediate
setImmediate
finish reading time: 19
...
setImmediate
setImmediate
```
