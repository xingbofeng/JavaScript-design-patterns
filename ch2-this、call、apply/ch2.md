# this、call、apply
## 2.1 this
### this的指向
* 当函数作为对象的方法被调用时，`this` 指向该对象

```javascript
var obj = {
  a: 1,
  getA: function() {
    alert(this === obj); // true
  }
};
obj.getA();
```

* 作为普通函数调用,`this` 总是指向全局对象。在浏览器的 `JavaScript` 里，这个全局对象是 `window` 对象。

```javascript
window.name = 'globalName';
var getName = function() {
  return this.name;
};
console.log(getName()); // globalName
```
或者：

```javascript
window.name = 'globalName';
var myObject = {
  name: 'sven',
  getName: function() {
    return this.name;
  }
};
var getName = myObject.getName; // 拿到这个函数的拷贝
console.log(getName()); // globalName
```

```javascript
document.getElementById('div1').onclick = function() {
  var that = this; // 保存div的引用
  var callback = function() { // 这个函数是指向window
    console.log(that.id); // 'div1'
  };
  callback();
}
```

* 构造器调用：用`new`运算符调用函数时，该函数总会返回一个对象，通常情况下，构造器里的 this 就指向返回的这个对象。

```javascript

var MyClass = function() {
  this.name = 'sven';
};
var obj = new MyClass();
console.log(obj.name); // 输出:sven
```

但用`new`调用构造器时，还要注意一个问题，如果构造器显式地返回了一个`object`类型的对
象，那么此次运算结果最终会返回这个对象，而不是我们之前期待的`this`。

```javascript
var MyClass = function() {
  this.name = 'sven';
  return { // 显式地返回一个对象
    name: 'anne'
  }
};
var obj = new MyClass();
console.log(obj.name); // 输出:anne
```

* `Function.prototype.call`或`Function.prototype.apply`调用
作用就是强行改变`this`

```javascript
var obj1 = {
  name: 'sven',
  getName: function() {
    return this.name;
  }
};
var obj2 = {
  name: 'anne'
};
console.log(obj1.getName()); // 输出: sven
console.log(obj1.getName.call(obj2)); // 输出:anne
```

### 丢失的this
```html
<html>
<body>
  <div id="div1">我是一个 div</div>
</body>
<script>
  var getId = document.getElementById;
  getId('div1');
</script>
</html>
```

在`Chrome`、`Firefox`、`IE10`中执行过后就会发现，这段代码抛出了一个异常。这是因为许多引擎的 `document.getElementById`方法的内部实现中需要用到`this`。这个`this`本来被期望指向`document`，当 `getElementById`方法作为`document`对象的属性被调用时，方法内部的`this`确实是指向`document`的。

```javascript
document.getElementById = (function(func) {
  return function() {
    return func.apply(document, arguments);
  }
})(document.getElementById);
```

## 2.2 call和apply
### call和apply的区别
* `apply`接受两个参数，第一个参数指定了函数体内`this`对象的指向，第二个参数为一个带下 标的集合，这个集合可以为**数组**，也可以为**类数组**，`apply`方法把这个集合中的元素作为参数传递给被调用的函数。
* `call`调用一个函数, 其具有一个指定的this值和分别地提供的参数(参数的列表)。

### call和apply的用途
* 改变`this`指向
* `Function.prototype.bind`

模拟一个简化版`bind`

```javascript
Function.prototype.bind = function(context) {
  var self = this; // 保存原函数
  var args = Array.prototype.slice.call(arguments); // 拿到参数
  return function() {
    return self.apply(context, args);
  }
}
var obj = {
  name: 'sven'
};

var func = function() {
  console.log(this.name);
}.bind(obj);

func();
```
通常我们还会把它实现得稍微复杂一点，使得可以往`func`函数中预先填入一些参数

```javascript
Function.prototype.bind = function() {
  var self = this, // 保存原函数
    context = [].shift.call(arguments), // 新的this是第一个参数
    args = [].slice.call(arguments); // 剩余的参数转成数组
  // 返回一个新的函数
  return function() {
    // 执行新的函数的时候，会把之前传入的 context 当作新函数体内的 this
    // 并且组合两次分别传入的参数，作为新函数的参数
    return self.apply(context, [].concat.call(args, [].slice.call(arguments)));
  }
};

var obj = {
  name: 'sven'
};

var func = function(a, b, c, d) {
  console.log(this.name); // sven
  console.log([a, b, c, d]) // [1, 2, 3, 4]
}.bind(obj, 1, 2); // 可以预传参数

func(3, 4);
```

### 借用方法
#### 实现继承

```javascript
var Parent = function(name) {
  this.name = name;
}

var Child = function() {
  Parent.apply(this, arguments); // 调用Parent构造函数实现继承
}

Child.prototype.getName = function() {
  return this.name;
}

var counterxing = new Child('counterxing');

console.log(counterxing.getName()); // counterxing
```

#### 操作类数组对象

```javascript
(function() {
  Array.prototype.push.call(arguments, 3);
  console.log(arguments); // [1,2,3]
})(1, 2);
```

类数组转化为数组：

```javascript
console.log(Array.prototype.slice.call(document.getElementsByTagName('div')) instanceof Array); // true
```