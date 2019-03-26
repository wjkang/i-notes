```js
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.toString = function() {
            return '(' + this.x + ', ' + this.y + ')';
        }
    }
    static staticMethod() {}
    toString() {
        return '(' + this.x + ', ' + this.y + ')';
    }
}
```
babel编译后：
```js
var Point =
  /*#__PURE__*/
  (function() {
    "use strict";

    function Point(x, y) {
      this.x = x;
      this.y = y;

      this.toString = function() {
        return "(" + this.x + ", " + this.y + ")";
      };
    }

    Point.staticMethod = function staticMethod() {};

    var _proto = Point.prototype;

    _proto.toString = function toString() {
      return "(" + this.x + ", " + this.y + ")";
    };

    return Point;
  })();
```

>使用的是es2015-loose模式

### 静态属性

ES6不支持静态属性的写法，需要使用babel插件`babel-plugin-transform-class-properties`

```js
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.toString = function() {
            return '(' + this.x + ', ' + this.y + ')';
        }
    }
    static staticMethod() {}
    static staticProperty = true;
    boundFunction = () = >{
        console.log(this);
    }
    a = 1;
    toString() {
        return '(' + this.x + ', ' + this.y + ')';
    }
}
```
babel编译后：
```js
var Point =
  /*#__PURE__*/
  (function() {
    "use strict";

    function Point(x, y) {
      var _this = this;

      this.boundFunction = function() {
        console.log(_this);
      };

      this.a = 1;
      this.x = x;
      this.y = y;

      this.toString = function() {
        return "(" + this.x + ", " + this.y + ")";
      };
    }

    Point.staticMethod = function staticMethod() {};

    var _proto = Point.prototype;

    _proto.toString = function toString() {
      return "(" + this.x + ", " + this.y + ")";
    };

    return Point;
  })();

Point.staticProperty = true;
```

此插件支持如下的写法：
```js
boundFunction = () = >{
    console.log(this);
}
a = 1;
```
!>此种写法，`bind`，`call`，`apply`无法修改对象方法内this的指向。

!>转ES5后方法直接挂在实例上，并不是类的原型上。

或者使用插件`@babel/plugin-proposal-class-properties`：

babel编译后：
```js
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

var Point =
  /*#__PURE__*/
  (function() {
    "use strict";

    function Point(x, y) {
      var _this = this;

      _defineProperty(this, "boundFunction", function() {
        console.log(_this);
      });

      _defineProperty(this, "a", 1);

      this.x = x;
      this.y = y;

      this.toString = function() {
        return "(" + this.x + ", " + this.y + ")";
      };
    }

    Point.staticMethod = function staticMethod() {};

    var _proto = Point.prototype;

    _proto.toString = function toString() {
      return "(" + this.x + ", " + this.y + ")";
    };

    return Point;
  })();

_defineProperty(Point, "staticProperty", true);
```
!>Class不支持像ES5一样使用闭包实现私有方法和私有属性。