>当Node.js启动时会初始化event loop, 每一个event loop都会包含按如下顺序六个循环阶段

![](../img/2019-05-28-00-02-27.png)

* **timers 阶段**: 这个阶段执行setTimeout(callback) and setInterval(callback)预定的callback;
* **I/O callbacks 阶段**: 执行除了 close事件的callbacks、被timers(定时器，setTimeout、setInterval等)设定的callbacks、setImmediate()设定的callbacks之外的callbacks;
* **idle, prepare 阶段**: 仅node内部使用;
* **poll 阶段**: 获取新的I/O事件, 适当的条件下node将阻塞在这里;
* **check 阶段**: 执行setImmediate() 设定的callbacks;
* **close callbacks 阶段**: 比如socket.on(‘close’, callback)的callback会在这个阶段执行.
