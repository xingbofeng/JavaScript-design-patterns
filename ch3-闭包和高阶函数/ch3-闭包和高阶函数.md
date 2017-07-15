# 闭包和高阶函数
## 3.1闭包
### 3.1.1变量作用域
### 3.1.2变量生命周期
```javascript
var func = function() {
  var a = 1;
  return function() {
    a++;
    console.log(a);
  }
}

var f = func();
f(); // 2
f(); // 3
f(); // 4
f(); // 5
```

当执行`var f = func()`时，`f`返回了一个匿名函数的引用，它可以访问到`func()`被调用时产生的环境。而局部变量`a`会一直在这个环境中。

同样的例子：

```javascript
var nodes = document.getElementsByTagName('div');
for (var i = 0; i < nodes.length; i++) {
  nodes[i].onclick = function() {
    console.log(i); // 全是5
  }
}
```

因为`onclick`事件异步触发，而事件触发的时候，`i`已经变成了5，所以输出全是5。

解决办法就是创建一个作用域。(`let` / `自执行匿名函数`)

### 3.1.3闭包的更多作用
#### 封装变量
闭包可以帮助把一些不需要在全局的变量封装成**私有变量**。

```javascript
var mult = function() {
  var a = 1;
  for (var i = 0; i < arguments.length; i++) {
    a *= arguments[i];
  }
  return a;
}
```

加入缓存机制：

```javascript
var mult = (function() {
  var cache = {}; // 变量仅仅在mult函数中被使用，不应在全局作用下声明
  return function() {
    var args = Array.prototype.join.call(arguments, ',');
    if (cache[args]) {
      return cache[args];
    }
    var a = 1;
    for (var i = 0; i < arguments.length; i++) {
      a *= arguments[i];
    }
    return cache[args] = a;
  }
})();

console.log(mult(1, 2, 3)); // 6
console.log(mult(1, 2, 3)); // 6
```

#### 延续局部变量的寿命
### 3.1.4闭包和面向对象设计
```javascript
var extent = function() {
  var value = 0;
  return {
    call: function() {
      value++;
      console.log(value);
    }
  };
}

var extent = extent();
extent.call(); // 1
extent.call(); // 2
extent.call(); // 3
```

如果换成面向对象的写法，就是：

```javascript
var extent = function() {
  return {
    value: 0,
    call: function() {
      this.value++;
      console.log(this.value);
    }
  };
}

var extent = extent();
extent.call(); // 1
extent.call(); // 2
extent.call(); // 3
```

## 3.2高阶函数
* 函数作为参数传递
* 函数作为返回值输出

### AOP(面向切面编程)
AOP的主要作用是把一些跟核心业务逻辑模块无关的功能抽离出来，这些跟业务逻辑模块无关的功能通常包括：日志统计、安全控制、异常处理等。把这些功能抽离出来之后，再通过“动态织入”的方式掺入业务逻辑模块中。

```javascript
Function.prototype.before = function(beforefn) {
  var self = this; // 保存原函数的引用
  return function() {
    beforefn.apply(this, arguments); // 先执行新函数，修正this
    return self.apply(this, arguments); // 再执行原函数
  }
}

Function.prototype.after = function(afterfn) {
  var self = this;
  return function() {
    var ret = self.apply(this, arguments); // 先执行原函数
    afterfn.apply(this, arguments); // 再执行新函数
    return ret;
  }
}

var func = function() {
  console.log(2);
}

func = func.before(function() {
  console.log(1);
}).after(function() {
  console.log(3);
});
func(); // 依次输出1，2，3
```

这种使用AOP的方式来给函数添加职责，也是`JavaScript`语言中一种特别的装饰者模式的实现。

### currying
`currying`又称部分求值。一个`currying`的函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到函数真正需要求值的时候，之前传入的所有参数都会被一次性用于求值。

```javascript
var monthlyCost = 0;

var cost = function() {
  monthlyCost += money;
}

cost(100);
cost(200);
cost(300);

console.log(monthlyCost); // 600
```

换为`currying`写法：

```javascript
var currying = function(fn) {
  var args = [];

  return function() {
    if (arguments.length === 0) {
      return fn.apply(this, args); // 没传参数时，调用这个函数
    } else {
      [].push.apply(args, arguments); // 传入了参数，把参数保存下来
      return arguments.callee; // 返回这个函数的引用
    }
  }
}

var cost = (function() {
  var money = 0;
  return function() {
    for (var i = 0; i < arguments.length; i++) {
      money += arguments[i];
    }
    return money;
  }
})();

var cost = currying(cost);

cost(100); // 传入了参数，不真正求值
cost(200); // 传入了参数，不真正求值
cost(300); // 传入了参数，不真正求值

console.log(cost()); // 求值并且输出600
```

### uncurrying
在`JavaScript`中，我们调用对象的某个方法时，其实不用关心该对象原本是否被设计为拥有这个方法，这是动态类型语言的特点。

用什么样的办法可以让对象去借用一个原本不属于它的方法？(`call / apply`)

```javascript
var obj1 = {
  name: 'counterxing'
};

var obj2 = {
  getName: function() {
    return this.name;
  }
};

console.log(obj2.getName.call(obj1)); // counterxing
```

使用`uncurrying`实现借用方法！泛化此过程：

```javascript
Function.prototype.uncurrying = function() {
  var self = this;
  return function() {
    var obj = Array.prototype.shift.call(arguments); // this
    return self.apply(obj, arguments); // 调用原函数，this已经出队了，剩下的是参数
  }
}

var push = Array.prototype.push.uncurrying();
(function() {
  push(arguments, 4);
  console.log(arguments);
})(1, 2, 3); // [1, 2, 3, 4]
```

```javascript
Function.prototype.uncurrying = function() {
  var self = this;
  return function() {
    var obj = Array.prototype.shift.call(arguments); // this
    return self.apply(obj, arguments); // 调用原函数，this已经出队了，剩下的是参数
  }
}

for (var i = 0, fn, ary = ['push', 'shift', 'forEach']; fn = ary[i++]; ) {
  Array[fn] = Array.prototype[fn].uncurrying();
}

var obj = {
  'length': 3,
  '0': 1,
  '1': 2,
  '2': 3,
};

Array.push(obj, 4);
console.log(obj.length);

var first = Array.shift(obj);
console.log(first);
console.log(obj);

Array.forEach(obj, function(i, n) {
  console.log(n);
});
```

### 函数节流
### 分时函数
### 惰性加载