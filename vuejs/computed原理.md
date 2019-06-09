### initComputed

`initComputed` 方法中，每个计算属性对应一个 `Watcher` 实例

> 为计算属性实例化 `Watcher` 的时候，没有立即进行求值。`this.value = this.lazy ? undefined : this.get();`

```js
function initComputed(vm, computed) {
  const watchers = (vm._computedWatchers = Object.create(null));
  for (const key in computed) {
    const userDef = computed[key];
    const getter = typeof userDef === "function" ? userDef : userDef.get;
    watchers[key] = new Watcher(vm, getter || function(), function() {}, { lazy: true });
    defineComputed(vm, key, userDef);
  }
}

function defineComputed(target, key, userDef) {
  sharedPropertyDefinition.get = createComputedGetter(key);
  sharedPropertyDefinition.set = function() {};
  Object.defineProperty(target, key, sharedPropertyDefinition);
}

function createComputedGetter(key) {
  return function computedGetter() {
    const watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate();
      }
      if (Dep.targetWatcher) {
        watcher.depend();
      }
      return watcher.value;
    }
  };
}
```
计算属性逻辑函数作为 `getter` 传入 `Watcher`

> `evaluate` 将 `dirty` 改为 false ,`update` 将 `dirty` 改为 true (重新求值后)

`computedGetter` 作为计算属性的 `get` 劫持逻辑

当访问计算属性的时候，`get` 劫持逻辑触发，第一次访问的话需要 `evaluate` 求值。

> 根据 `return watcher.value` 可以看出为什么计算属性内逻辑必须是同步的

执行 `evaluate` 可能会访问到其它属性，触发相应属性的 `get` 劫持逻辑。

**`evaluate` 调用 `get`会将当前 `Watcher` 实例设为激活，那么计算属性函数逻辑内访问到的其它属性，在触发他们的 `get` 劫持逻辑的时候，是可以获取到当前激活的`Watcher` 实例的**。

**所以依赖的属性对应的 `Dep` 实例的 `depend` 方法会被调用，也就是计算属性对应的 `Watcher` 实例的 `addDep` 方法会被调用多次，最终的结果就是：每个依赖的属性对应的 `Dep` 实例的 `subs` 中都有计算属性对应的 `Watcher` 实例，更新依赖属性的时候，会执行 `update`方法，将`dirty`改为 true（并不会重新求值，下次访问计算属性的时候才会重新求值）,如果依赖属性没有变更那么计算属性一直使用上次的计算值 **

>不清楚什么情况下，会执行
```js
if (Dep.targetWatcher) {
  watcher.depend();
}
```
>因为当执行完 `evaluate`，`Dep.targetWatcher` 又会设置为 null

计算属性的 `watcher` 中，保存有相应依赖属性的 `dep`。计算属性重新计算的时候（被访问到了），会重新收集依赖（依赖属性的 `dep`收集当前计算属性的`watcher`），计算完之后还会移除没有的依赖（计算属性可能不依赖某个属性了，对应属性的`dep`删除计算属性的`watcher`）。

>当计算属性不再被访问（不再执行任何关于计算属性对应的 `watcher`的逻辑），依赖并没有被移除（之前依赖属性的 `dep`中还收集计算属性的`watcher`）

**计算属性依赖的属性没有绑定在界面上，更新属性的时候会不会触发组件的更新检查？**
* 首次渲染的时候（执行 `mountComponent`），初始化负责组件 `update` 的 `watcher`,并且`watcher`内直接执行`get`方法（计算属性的 `watcher`不会执行）。
* `updateComponent`对应的`watcher`加入`targetStack`。
* 第一次访问计算属性，执行`evaluate`，计算属性对应的`watcher`加入`targetStack`（计算计算属性的时候，关于依赖属性的访问逻辑前面已经分析过）。
* `evaluate`执行完，计算属性对应的`watcher`从`targetStack`移除，`Dep.target`对应`updateComponent`的`watcher`
* 执行`watcher.depend()`(计算属性对应的`watcher`)。
>```js
 depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }
 ```
* `deps`为计算属性所依赖的属性对应的 `dep`。

>```js
 depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
 ```

 * 计算属性所依赖的属性对应的 `dep`的`subs`会添加`updateComponent`的`watcher`。
 >`watcher`的`deps`会添加属性对应的 `dep`。（也就是直接访问属性或者通过计算属性间接访问，都会如此）

所以，计算属性依赖的属性即使没有绑定在界面上，更新属性的时候依然会触发组件的更新检查。

**当计算属性或依赖属性一直被访问，依赖会一直存在着**：

* 组件更新的时候，`Dep.target`对应`updateComponent`的`watcher`。
* 绑定计算属性，执行
```js
if (watcher.dirty) {
  watcher.evaluate()
}
if (Dep.target) {
  watcher.depend()
}
return watcher.value
```
所以依赖会一直收集着。

**当计算属性不再被访问或者计算属性中某个依赖属性不再被访问，更新属性的时候是否会触发组件的更新检查（依赖属性对应的`dep`的`subs`是否已经将相应的`watcher`移除）**

* 某次更新后，达到某种临界值，计算属性不再被访问或者计算属性中某个依赖属性不再被访问。
* 更新依赖属性，还会触发组件更新。
* 计算属性不再被访问
 * `watcher`的`newDeps`不再收集有依赖属性的`dep`。
 * 执行`cleanupDeps`，将`watcher`从依赖属性的`dep`的`subs`移除。
 * 更新依赖属性不再触发组件更新。
* 某个依赖属性不再被访问
 * 如果需要执行`evaluate`，计算属性的`watcher`不再收集相应的依赖属性的 `dep`，也就是相应`dep`的`subs`移除计算属性的`watcher`（更新依赖属性不再触发计算属性的`watcher`的`update`）。
 * 执行`watcher.depend()`，因为`watcher`不再收集相应的依赖属性的 `dep`，所以依赖属性的`dep` 也不会再加到`Dep.target`的`newDeps`中。
 * 执行`cleanupDeps`后，相应`dep`的`subs`移除组件更新的`watcher`（更新依赖属性不再触发组件更新）


>`watcher.depend()`的作用是将计算属性的`watcher`的`deps`加入`Dep.target`所指向的`watcher`（正常情况下就是负责更新组件的`watcher`）的`deps`中。


