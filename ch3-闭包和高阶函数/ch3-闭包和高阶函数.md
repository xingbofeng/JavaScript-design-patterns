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