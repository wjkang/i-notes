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

>当计算属性不再被访问，依赖并没有被移除（之前依赖属性的 `dep`中还收集计算属性的`watcher`）