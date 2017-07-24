# 发布-订阅模式(观察者模式)
* 发布—订阅模式可以广泛应用于异步编程中，这是一种替代传递回调函数的方案。 比如，我们可以订阅`ajax`请求的`error`、`success`等事件。或者如果想在动画的每一帧完成之后做一些事情，那我们可以订阅一个事件，然后在动画的每一帧完成之后发布这个事件。在异步编程中使用发布—订阅模式，我们就无需过多关注对象在异步运行期间的内部状态，而只需要订阅感兴趣的事件发生点。
* 发布—订阅模式可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口。发布—订阅模式让两个对象松耦合地联系在一起，虽然不太清楚彼此的细节，但这不影响它们之间相互通信。当有新的订阅者出现时，发布者的代码不需要任何修改;同样发布者需要改变时，也不会影响到之前的订阅者。只要之前约定的事件名没有变化，就 可以自由地改变它们。

```javascript
// 发布订阅
window.$.ajax({
  // url
  // method
  success: function(res) {
    Event.emit(res);
  },
  error: function(err) {
    Event.emit(res);
  }
});

// 两对象耦合
window.$.ajax({
  // url
  // method
  success: function(res) {
    result.something(res);
  },
  error: function(err) {
    result.something(res);
  }
});
```

## 8.3 DOM事件
绑定DOM事件就是一种发布订阅模式。
监控用户点击`document.body`的动作，但是我们没办法预知用户将在什么时候点击。所以我们订阅`document.body`上的`click`事件，当`body`节点被点击时，`body`节点便会向订阅者发布这个消息。这很像购房的例子，购房者不知道房子什么时候开售，于是他在订阅消息后等待售楼处发布消息。

```javascript
document.body.addEventListener('click', function() {
  console.log('click!');
}, false);
document.body.click(); // 模拟用户点击
```

我们还可以随意增加或者删除订阅者，增加任何订阅者都不会影响发布者代码的编写:

```javascript
document.body.addEventListener('click', function() {
  console.log('click2!');
}, false);
document.body.addEventListener('click', function() {
  console.log('click3!');
}, false);
document.body.addEventListener('click', function() {
  console.log('click4!');
}, false);
document.body.click(); // 模拟用户点击
```

## 8.4 自定义事件
如何一步步实现发布—订阅模式？

* 指定好发布者;
* 发布者有一个缓存列表，里面存放了回调函数，以便发布后通知订阅者;
* 发布消息的时候遍历缓存列表，依次触发订阅者的回调;

```javascript
var salesOffices = {}; // 定义售楼处

salesOffices.clientList = []; // 缓存列表，存放订阅者的回调函数

salesOffices.listen = function(fn) { // 增加订阅者 
  this.clientList.push(fn); // 订阅的消息添加进缓存列表
};

salesOffices.trigger = function() { // 发布消息 
  for (var i = 0, fn; fn = this.clientList[i++];) {
    fn.apply(this, arguments); // arguments是发布消息时带上的参数
  }
};
```

```javascript
// 小明订阅消息
salesOffices.listen(function(price, squareMeter) {
  console.log('价格：' + price);
  console.log('面积：' + squareMeter);
});

// 小红订阅消息
salesOffices.listen(function(price, squareMeter) {
  console.log('价格：' + price);
  console.log('面积：' + squareMeter);
});

// 发布消息
salesOffices.trigger(2000000, 88); // 输出:200万，88平方米
salesOffices.trigger(3000000, 110); // 输出:300万，110平方米
```

excuse me? 输出了两次？

这里还存在一些问题。我们看到订阅者接收到了发布者发布的每个消息，虽然小明只想买88平方米的房子，但是发布者把110平方米的信息也推送给了小明，这对小明来说是不必要的困扰。

所以我们有必要增加一个标示`key`，让订阅者只订阅自己感兴趣的消息。

```javascript
var salesOffices = {}; // 定义售楼处(发布者)

salesOffices.clientList = {}; // 缓存列表，存放订阅者的回调函数

salesOffices.listen = function(key, fn) {
  // 如果还没有订阅过此类消息，给该类消息创建一个缓存列表
  if (!this.clientList[key]) {
    this.clientList[key] = [];
  }
  // 订阅的消息添加进消息缓存列表
  this.clientList[key].push(fn);
};

salesOffices.trigger = function() { // 发布消息
  var key = Array.prototype.shift.call(arguments); // 取出消息类型，第一个参数
  var fns = this.clientList[key]; // 拿到消息列表里的函数
  // 如果没有订阅该消息，则返回
  if (!fns || fns.length === 0) {
    return false;
  }
  for (var i = 0, fn; fn = fns[i++];) {
    fn.apply(this, arguments); // arguments 是发布消息时附送的参数
  }
};

// 订阅消息
salesOffices.listen('squareMeter88', function(price) {
  console.log('价格= ' + price); // 输出: 2000000
});
salesOffices.listen('squareMeter110', function(price) {
  console.log('价格= ' + price); // 输出: 3000000
});

// 发布消息
salesOffices.trigger('squareMeter88', 2000000); // 发布 88 平方米房子的价格
salesOffices.trigger('squareMeter110', 3000000); // 发布 110 平方米房子的价格
```

## 8.5 通用实现
可移植的发布订阅模式。

```javascript
var event = {
  clientList: [],
  listen: function(key, fn) {
    if (!this.clientList[key]) {
      this.clientList[key] = [];
    }
    // 订阅的消息添加进缓存列表 
    this.clientList[key].push(fn);
  },
  trigger: function() {
    var key = Array.prototype.shift.call(arguments);
    var fns = this.clientList[key];
    if (!fns || fns.length === 0) { // 如果没有绑定对应的消息
      return false;
    }
    for (var i = 0, fn; fn = fns[i++];) {
      fn.apply(this, arguments); // (2) // arguments 是 trigger 时带上的参数
    }
  }
};

// **再定义一个 installEvent 函数，这个函数可以给所有的对象都动态安装发布—订阅功能**
var installEvent = function(obj) {
  for (var i in event) {
    obj[i] = event[i];
  }
};

var salesOffices = {}; // 定义发布者(售楼处)
installEvent(salesOffices);

// 小明订阅消息
salesOffices.listen('squareMeter88', function(price) {
  console.log('价格= ' + price);
});

// 小红订阅消息
salesOffices.listen('squareMeter100', function(price) {
  console.log('价格= ' + price);
});
salesOffices.trigger('squareMeter88', 2000000); // 输出:2000000
salesOffices.trigger('squareMeter100', 3000000); // 输出:3000000
```

## 8.6 取消订阅
```javascript
event.remove = function(key, fn) {
  var fns = this.clientList[key];
  // 如果 key 对应的消息没有被人订阅，则直接返回
  if (!fns) {
    return false;
  }
  // 如果没有传入具体的回调函数，表示需要取消key对应消息的所有订阅
  // 如果传入参数则取消对应的订阅
  if (!fn) {
    fns && (fns.length = 0);
  } else {
    for (var l = fns.length - 1; l >= 0; l--) { // 反向遍历订阅的回调函数列表 
      var _fn = fns[l];
      if (_fn === fn) {
        fns.splice(l, 1);
        // 删除订阅者的回调函数
      }
    }
  }
};

// 动态安装发布—订阅功能
var salesOffices = {};
var installEvent = function(obj) {
  for (var i in event) {
    obj[i] = event[i];
  }
}
installEvent(salesOffices);

// 小明订阅消息
salesOffices.listen('squareMeter88', fn1 = function(price) {
  console.log('价格= ' + price);
});

// 小红订阅消息
salesOffices.listen('squareMeter88', fn2 = function(price) {
  console.log('价格= ' + price);
});

// 删除小明的订阅
salesOffices.remove('squareMeter88', fn1);

salesOffices.trigger('squareMeter88', 2000000); // 输出:2000000
```

## 8.7 真实的例子
## 8.8 全局的发布订阅
虽然基本实现了发布订阅模式，但是现在还有一些缺陷：

* 我们给每个发布者对象都添加了`listen`和`trigger`方法，以及一个缓存列表`clientList`，这其实是一种资源浪费。
* 必须知道发布者的名字叫`salesOffices`，一旦想订阅另外一个发布者，我们得再粘一次代码。

发布—订阅模式可以用一个全局的`Event`对象来实现，订阅者不需要了解消息来自哪个发布者，发布者也不知道消息会推送给哪些订阅者，`Event`作为一个类似`中介者`的角色，把`订阅者`和`发布者`联系起来

使用一个全局的`Event`对象（唯一一个）：

```javascript
var Event = (function() {
  var clientList = {},
    listen, trigger, remove;
  listen = function(key, fn) {
    if (!clientList[key]) {
      clientList[key] = [];
    }
    clientList[key].push(fn);
  };
  trigger = function() {
    var key = Array.prototype.shift.call(arguments);
    var fns = clientList[key];
    if (!fns || fns.length === 0) {
      return false;
    }
    for (var i = 0, fn; fn = fns[i++];) {
      fn.apply(this, arguments);
    }
  };
  remove = function(key, fn) {
    var fns = clientList[key];
    if (!fns) {
      return false;
    }
    if (!fn) {
      fns && (fns.length = 0);
    } else {
      for (var l = fns.length - 1; l >= 0; l--) {
        var _fn = fns[l];
        if (_fn === fn) {
          fns.splice(l, 1);
        }
      }
    }
  };
  return {
    listen: listen,
    trigger: trigger,
    remove: remove
  };
})();

// 此时我们都使用全局的Event来进行发布订阅

// 订阅
Event.listen('squareMeter88', function(price) {
  console.log('价格= ' + price);
});

// 发布
Event.trigger('squareMeter88', 2000000);
```

## 8.9 模块间通信

```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
</head>
<body>
  <button id="count">点我</button>
  <div id="show"></div>
  <script type="text/javascript">
    var a = (function() {
      var count = 0;
      var button = document.getElementById('count');
      button.onclick = function() {
        Event.trigger('add', count++);
      }
    })();
    var b = (function() {
      var div = document.getElementById('show');
      Event.listen('add', function(count) {
        div.innerHTML = count;
      });
    })();
  </script>
</body>
</html>
```

## 8.10 必须先订阅再发布吗
不一定！

建立一个存放离线事件的堆栈，当事件发布的时候，如果此时还没有订阅者来订阅这个事件，我们暂时把发布事件的动作包裹在一个函数里，这些包装函数将被存入堆栈中，等到终于有对象来订阅此事件的时候，我们将遍历堆栈并且依次执行这些包装函数，也就是重新发布里面的事件。当然离线事件的生命周期只有一次，就像`QQ`的未读消息只会被重 新阅读一次，所以刚才的操作我们只能进行一次。

## 小结
发布—订阅模式的优点非常明显，一为时间上的解耦，二为对象之间的解耦。它的应用非常广泛，既可以用在异步编程中，也可以帮助我们完成更松耦合的代码编写。发布—订阅模式还可以用来帮助实现一些别的设计模式，比如中介者模式。从架构上来看，无论是 MVC 还是 MVVM， 都少不了发布—订阅模式的参与，而且`JavaScript`本身也是一门基于事件驱动的语言。

当然，发布—订阅模式也不是完全没有缺点（浪费内存）。创建订阅者本身要消耗一定的时间和内存，而且当你订阅一个消息后，也许此消息最后都未发生，但这个订阅者会始终存在于内存中。另外，发布—订阅模式虽然可以弱化对象之间的联系，但如果过度使用的话，对象和对象之间的必要联系也将被深埋在背后，会导致程序难以跟踪维护和理解。特别是有多个发布者和订阅者（b订阅a的消息并发布给c）嵌套到一起的时候，要跟踪一个`bug`不是件轻松的事情。
