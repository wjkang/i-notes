### 求最小值

```js
var numbers = [1,2,3,4,5]; 
Math.min(1,2,3,4,5);
Math.min.apply(null, numbers);   
Math.min.call(null, 1,2,3,4,5);
Math.min.call(null, ...numbers);
```
### 判断类型
```js
Object.prototype.toString.call(1);
Object.prototype.toString.call("1");
Object.prototype.toString.call(true);
Object.prototype.toString.call({a:1});
Object.prototype.toString.call([1]);
Object.prototype.toString.call(new Date);
Object.prototype.toString.call(/a-z/);
Object.prototype.toString.call(new Number(1));
Object.prototype.toString.call(new String("1"));
Object.prototype.toString.call(new Boolean(true));
Object.prototype.toString.call(new Object);
Object.prototype.toString.call(undefined);
Object.prototype.toString.call(null);
Object.prototype.toString.call(NaN);
Object.prototype.toString.call(JSON);
```
!>前提toString方法没有被重写。

`(new Number(1)).toString()`的结果为"1"，因为在`Number`内`toString`已经被重写。

### 模拟实现

#### 实现调用
```js
Function.prototype.apply2 = function(context) {
    context.fn = this; 	
    context.fn();		
    delete context.fn;
}
var data={
    value:1
}
function log(){
    console.log(this.value);
}
log.apply2(data);
```
* this指向log函数。

* context.fn指向log函数。

* context.fn就是data.fn。

* data.fn()就是在data上下文执行log函数，this指向data。

* 删除data上的fn。

#### 支持参数

```js
Function.prototype.apply2 = function(context,arr) {
    context.fn = this; 	
    if(!arr){
        context.fn();
    }else{
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        eval('context.fn(' + args +')');	
    }	
    delete context.fn;
}
var data={
    value:1
}
function log(str1,str2){
    console.log(this.value,str1,str2);
}
log.apply2(data,["A","B"]);
```
* 模拟apply，所以第二个参数传入一个数组。

* 没有传入参数，直接调用context.fn()。

* 需要以log("A","B")的方式调用函数，因为无法确定参数个数，需要使用eval。

!>考虑new Function能否替换eval。

#### 修正this，处理返回值

```js
Function.prototype.apply2 = function(context,arr) {
    context = context ? Object(context) : window;
    context.fn = this; 	
    var result;
    if(!arr){
        result = context.fn();
    }else{
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result =eval('context.fn(' + args +')');	
    }	
    delete context.fn;
    return result;
}
var data={
    value:1
}
function log(str1,str2){
    console.log(this.value,str1,str2);
}
log.apply2(data,["A","B"]);
```