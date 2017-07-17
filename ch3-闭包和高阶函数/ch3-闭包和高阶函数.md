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
* 函数去抖（`debounce`）：让一个函数在一定间隔内没有被调用时，才开始执行被调用方法。
* 函数节流（`throttle`）：是让一个函数无法在很短的时间间隔内连续调用，当上一次函数执行后过了规定的时间间隔，才能进行下一次该函数的调用。 函数节流的出发点，就是让一个函数不要执行得太频繁，减少一些过快的调用来节流。

举个例子：

想象每天上班大厦底下的电梯。把电梯完成一次运送，类比为一次函数的执行和响应。假设电梯有两种运行策略`throttle`和`debounce`，超时设定为15秒，不考虑容量限制。

* `throttle`策略的电梯。保证如果电梯第一个人进来后，15秒后准时运送一次，不等待。如果没有人，则待机。
* `debounce`策略的电梯。如果电梯里有人进来，等待15秒。如果又人进来，15秒等待重新计时，直到15秒超时，开始运送。

应用场景：

* 百度谷歌输入框
* `mousemove`与`resize`事件

```javascript
/**
 * 频率控制 返回函数连续调用时，func 执行频率限定为 次 / wait
 * 
 * @param  {function}   func      传入函数
 * @param  {number}     wait      表示时间窗口的间隔
 * @param  {object}     options   如果想忽略开始边界上的调用，传入{leading: false}。
 *                                如果想忽略结尾边界上的调用，传入{trailing: false}
 * @return {function}             返回客户调用函数   
 */
_.throttle = function(func, wait, options) {
  var context, args, result;
  var timeout = null;
  // 上次执行时间点
  var previous = 0;
  if (!options) options = {};
  // 延迟执行函数
  var later = function() {
    // 若设定了开始边界不执行选项，上次执行时间始终为0
    previous = options.leading === false ? 0 : _.now();
    timeout = null;
    result = func.apply(context, args);
    if (!timeout) context = args = null;
  };
  return function() {
    var now = _.now();
    // 首次执行时，如果设定了开始边界不执行选项，将上次执行时间设定为当前时间。
    // 否则previous这里是0，如果是0，remaining肯定小于0，则会执行func.apply(context, args);这条语句，执行业务逻辑
    if (!previous && options.leading === false) previous = now;
    // 延迟执行时间间隔
    var remaining = wait - (now - previous);
    context = this;
    args = arguments;
    // 延迟时间间隔remaining小于等于0，表示上次执行至此所间隔时间已经超过一个时间窗口
    // remaining大于时间窗口wait，表示客户端系统时间被调整过
    if (remaining <= 0 || remaining > wait) {
      // 清除定时器防止内存泄露
      // clearTimeout(timeout)后，timeout的值并不会清空，如果不设置为null，就不能根据timeout设置下次的timeout
      clearTimeout(timeout);
      timeout = null;
      // 把previous置为现在的时间戳(下次判断用)
      previous = now;
      // 调用func
      result = func.apply(context, args);
      if (!timeout) context = args = null;
      //如果延迟执行不存在，且没有设定结尾边界不执行选项
    } else if (!timeout && options.trailing !== false) {
      // 设定把timeout定时器，设定为remaining毫秒后推入定时器回调队列执行later回调函数
      timeout = setTimeout(later, remaining);
    }
    return result;
  };
};
```

```javascript
/**
 * 空闲控制 返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
 *
 * @param  {function} func        传入函数
 * @param  {number}   wait        表示时间窗口的间隔
 * @param  {boolean}  immediate   设置为ture时，调用触发于开始边界而不是结束边界
 * @return {function}             返回客户调用函数
 */
_.debounce = function(func, wait, immediate) {
  var timeout, args, context, timestamp, result;

  var later = function() {
    // 据上一次触发时间间隔
    var last = _.now() - timestamp;

    // 上次被包装函数被调用时间间隔last小于设定时间间隔wait
    // 重新设定定时器，不过时间更新了！
    if (last < wait && last > 0) {
      timeout = setTimeout(later, wait - last);
    } else {
      timeout = null;
      // 如果设定为immediate===true，因为开始边界已经调用过了此处无需调用
      if (!immediate) {
        // 执行业务逻辑
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      }
    }
  };

  return function() {
    context = this;
    args = arguments;
    timestamp = _.now();
    // 立即触发需要满足两个条件：immediate为true，且timeout不是null
    // 根据 timeout 是否为空可以判断是否是首次触发
    var callNow = immediate && !timeout;
    // 如果延时不存在，重新设定延时
    if (!timeout) timeout = setTimeout(later, wait);
    if (callNow) {
      // 达到立即触发的条件（第一次调用），调用业务逻辑函数
      result = func.apply(context, args);
      context = args = null;
    }

    return result;
  };
};
```
### 分时函数
### 惰性加载