```js
Function.prototype.bind = function(context) {
    var self = this; 
    return function () { 
        return self.apply(context); 
    }
}
```
```js
Function.prototype.bind = function (context) {

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1); 
    return function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply( context, args.concat(bindArgs) );
    }
}
```
