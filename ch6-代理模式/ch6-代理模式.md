# 代理模式

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

![](http://oczira72b.bkt.clouddn.com/17-8-6/93444678.jpg)

**关键：** 客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身 对象来控制对这个对象的访问，客户实际上访问的是替身对象。

## 最简单的代理模式

不使用代理模式，小明给A送花：

```javascript
var Flower = function() {};
var xiaoming = {
  sendFlower: function(target) {
    var flower = new Flower();
    target.receiveFlower(flower);
  }
};
var A = {
  receiveFlower: function(flower) {
    console.log('收到花 ' + flower);
  }
};
xiaoming.sendFlower(A);
```

使用代理模式，小明通过B给A送花：

```javascript
var Flower = function() {};
var xiaoming = {
  sendFlower: function(target) {
    var flower = new Flower();
    target.receiveFlower(flower);
  }
};
var B = {
  receiveFlower: function(flower) {
    A.receiveFlower(flower); // 把这里把花给A（调用A的方法）
  }
};
var A = {
  receiveFlower: function(flower) {
    console.log('收到花 ' + flower);
  }
};
xiaoming.sendFlower(B); // 把花给B
```

此处的代理模式毫无用处，它所做的只是把请求简单地转交给本体。但不管怎样，我们开始引入了代理，这是一个不错的起点。

改变故事的背景设定，B(经纪人)更了解A：

```javascript
var Flower = function() {};
var xiaoming = {
  sendFlower: function(target) {
    var flower = new Flower();
    target.receiveFlower(flower);
  }
};
var B = {
  receiveFlower: function(flower) {
    A.listenGoodMood(function() {
      A.receiveFlower(flower);
    });
  }
};
var A = {
  receiveFlower: function(flower) {
    // 监听 A 的好心情
    console.log('收到花 ' + flower);
  },
  listenGoodMood: function(fn) {
    setTimeout(function() { // 假设3秒之后 A 的心情变好
      fn();
    }, 3000);
  }
};

xiaoming.sendFlower(B);
```

## 保护代理和虚拟代理

* 保护代理：代理B可以帮助A过滤掉一些请求，比如送花的人中年龄太大的或者没有宝马的，这种请求就可以直接在代理 B 处被拒绝掉。
* 虚拟代理：现实中的花价格不菲，导致在程序世界里，`new Flower`也是一个代价昂贵的操作， 那么我们可以把`new Flower`的操作交给代理B去执行，代理B会选择在A心情好时再执行`new Flower`。

```javascript
var B = {
  receiveFlower: function(flower) {
    A.listenGoodMood(function() {
      var flower = new Flower();
      A.receiveFlower(flower);
    });
  }
};
```

## 图片预加载（虚拟代理）
在`Web`开发中，图片预加载是一种常用的技术，如果直接给某个`img`标签节点设置`src`属性， 由于图片过大或者网络不佳，图片的位置往往有段时间会是一片空白。常见的做法是先用一张`loading`图片占位，然后用异步的方式加载图片，等图片加载好了再把它填充到`img`节点里，这种 场景就很适合使用虚拟代理。

如果不使用图片预加载：

```javascript
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return {
    setSrc: function(src) {
      imgNode.src = src;
    }
  }
})();
myImage.setSrc('https://avatars3.githubusercontent.com/u/15172026?v=4&s=460');
```

我们把网速调至`5KB/s`，然后通过`MyImage.setSrc`给该`img`节点设置`src`，可以看到，在图片
被加载好之前，页面中有一段长长的空白时间。

现在开始引入代理对象`proxyImage`，通过这个代理对象，在图片被真正加载好之前，页面中
将出现一张占位图`loading`, 来提示用户图片正在加载。

```javascript
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  // 返回这个对象，具有setSrc方法，即设定图片的src属性
  return {
    setSrc: function(src) {
      imgNode.src = src;
    }
  }
})();
var proxyImage = (function() {
  var img = new Image;
  img.onload = function() {
    myImage.setSrc(this.src);
  }
  return {
    setSrc: function(src) {
      myImage.setSrc('https://www.baidu.com/img/bd_logo1.png'); // 初始情况设定logo
      img.src = src;
    }
  }
})();
proxyImage.setSrc('https://avatars3.githubusercontent.com/u/15172026?v=4&s=460');
```

## 代理的意义
实现一个小小的图片预加载功能，即使不需要引入任何模式也能办到，那么引入代理模式的好处究竟在哪里呢?

```javascript
var MyImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  var img = new Image;
  img.onload = function() {
    imgNode.src = img.src;
  };
  return {
    setSrc: function(src) {
      imgNode.src = 'https://www.baidu.com/img/bd_logo1.png';
      img.src = src;
    }
  }
})();
MyImage.setSrc('https://avatars3.githubusercontent.com/u/15172026?v=4&s=460');
```

**单一职责原则**

就一个类(通常也包括对象和函数等)而言，应该仅有一个引起它变化的原因。如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起它变化的原因可能会有多个。

（这里举个实际例子项目的例子）

`MyImage`对象除了负责给`img`节点设置`src`外，还要负责预加载图片。我们在处理其中一个职责时，有可能因为其强耦合性影响另外一个职责的实现。

更好的写法（代理和本体都提供`setSrc`方法，对用户而言不太友好）：

```javascript

// 用于创建img节点
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return function(src) {
    imgNode.src = src;
  }
})();
// 用于虚拟代理
var proxyImage = (function() {
  var img = new Image;
  img.onload = function() {
    myImage(this.src);
  }
  return function(src) {
    myImage('https://www.baidu.com/img/bd_logo1.png');
    img.src = src;
  }
})();
proxyImage('https://avatars3.githubusercontent.com/u/15172026?v=4&s=460');
```

## 虚拟代理实现合并HTTP请求

`open example.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <input type="checkbox" id="1"></input>1
  <input type="checkbox" id="2"></input>2
  <input type="checkbox" id="3"></input>3
  <input type="checkbox" id="4"></input>4
  <input type="checkbox" id="5"></input>5
  <input type="checkbox" id="6"></input>6
  <input type="checkbox" id="7"></input>7
  <input type="checkbox" id="8"></input>8
  <input type="checkbox" id="9"></input>9
</body>
</html>
```

接下来，给这些`checkbox`绑定点击事件，并且在点击的同时往另一台服务器同步文件:

```javascript
var synchronousFile = function(id) {
  console.log('开始同步文件，id 为: ' + id);
};
var checkbox = document.getElementsByTagName('input');
for (var i = 0, c; c = checkbox[i++];) {
  c.onclick = function() {
    if (this.checked === true) {
      synchronousFile(this.id);
    }
  }
};
```

当我们选中3个`checkbox`的时候，依次往服务器发送了3次同步文件的请求。频繁的网络请求将会带来相当大的开销。

通过一个代理函数`proxySynchronousFile`（单一职责原则）来收集一段时间之内的请求，最后一次性发送给服务器。

```javascript
var synchronousFile = function(id) {
  console.log('开始同步文件，id 为: ' + id);
};
// 代理类专门用来处理文件同步，接口请求
var proxySynchronousFile = (function() {
  var cache = [], // 保存一段时间内需要同步的 ID
    timer; // 定时器
  return function(id) {
      cache.push(id);
      if (timer) { // 保证不会覆盖已经启动的定时器
        return;
      }
      timer = setTimeout(function() {
        synchronousFile(cache.join(','));
        clearTimeout(timer); // 清空定时器
        timer = null;
        cache.length = 0; // 清空 ID 集合
      }, 2000);
    }
    // 2 秒后向本体发送需要同步的 ID 集合
})();

var checkbox = document.getElementsByTagName('input');
for (var i = 0, c; c = checkbox[i++];) {
  c.onclick = function() {
    if (this.checked === true) {
      proxySynchronousFile(this.id);
    }
  }
};
```

## miniConsole.js(跟虚拟代理实现图片加载道理一致)
## 缓存代理
缓存代理可以为一些开销大的运算结果提供暂时的存储，在下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。

计算乘积：

```javascript
var mult = function() {
 console.log('开始计算乘积');
 var a = 1;
 for (var i = 0, l = arguments.length; i < l; i++) {
   a = a * arguments[i]
 }
 return a;
};

mult(2, 3);
mult(2, 3, 4);
```

加入缓存代理

```javascript
var proxyMult = (function() {
  var cache = {};
  return function() {
    var args = Array.prototype.join.call(arguments, ',');
    if (args in cache) {
      return cache[args];
    }
    return cache[args] = mult.apply(this, arguments);
  }
})();
proxyMult(1, 2, 3, 4); // 输出:24
proxyMult(1, 2, 3, 4); // 输出:24
```

## 小结
在`JavaScript`开发中最常用的是虚拟代理和缓存代理。虽然代理模式非常有用，但我们在编写业务代码的时候，往往不需要去预先猜测是否需要使用代理模式。当真正发现不方便直接访问某个对象的时候，再编写代理也不迟。