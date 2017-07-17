# 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。
## 4.1实现单例模式
用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。

```javascript
var Singleton = function() {
  this.name = name;
  this.instance = null;
};

Singleton.prototype.getName = function() {
  return this.name;
};

Singleton.getInstance = function(name) {
  if (!this.instance) {
    this.instance = new Singleton(name);
  }
  return this.instance;
};

var a = Singleton.getInstance('counterxing');
var b = Singleton.getInstance('counter');

console.log(a === b); // true
```

**不好的地方**：创建对象只能通过`getInstance`创建，增加了这个类的不透明性。`Singleton`的使用者必须知道这是一个单例类，跟以往通过`new`方式来获取对象不同。

## 4.2透明的单例模式

```javascript
var createDiv = (function() {
  var instance;
  var createDiv = function(html) {
    if (instance) {
      return instance;
    }
    this.html = html;
    this.init();
    return instance = this;
  };
  createDiv.prototype.init = function() {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
  };
  return createDiv;
})();

var a = new createDiv('counter');
var b = new createDiv('counterxing');

console.log(a === b); // true
```

注：`createDiv`的构造函数实际上负责了两件事情。第一件事是创建对象和初始化`init`方法，第二件事是保证只有一个对象。

**单一职责原则**

假设某天我们需要使用这个类，在页面中创建多个`div`，即要使用这个类从单例类变成普通的可产生多个实例的类。我们必须得改写`createDiv`构造函数。这种修改会给我们带来不必要的烦恼。

## 4.3用代理实现单例模式
把创建对象和初始化与保证单例两件事情分开来做。

```javascript
// 创建对象
var createDiv = function(html) {
  this.html = html;
  this.init();
};

createDiv.prototype.init = function() {
  var div = document.createElement('div');
  div.innerHTML = this.html;
  document.body.appendChild(div);
};

// 保证单例
var proxySingletonCreateDiv = (function() {
  var instance;
  return function(html) {
    if (!instance) {
      instance = new createDiv(html);
    }
    return instance;
  }
})();

var a = new proxySingletonCreateDiv('counterxing');
var b = new proxySingletonCreateDiv('counter');

console.log(a === b);
```
## 4.4 JavaScript中的单例模式
单例模式的核心是：**确保只有一个实例，并提供全局访问。**

全局变量不是单例模式，但在`JavaScript`开发中，我们经常把全局变量当作单例来使用。

```javascript
var $ = require('jquery');
```

要减少全局变量的使用，即使需要，也要把它的污染降到最低。

* 使用命名空间
* 使用自执行匿名函数封装私有变量

## 4.5惰性单例
惰性单例是指在需要的时候才创建对象实例。如最开始那个例子`instance`实例对象总是在我们调用`Singleton.getInstance`的时候才被创建，而不是在页面加载好的时候就创建。

## 4.6通用的惰性单例
如果我们又需要创建页面中唯一的`iframe`，或者`script`标签，用于跨域请求数据。

```javascript
var createIframe = (function() {
  var iframe;
  return function() {
    if (!iframe) {
      iframe = document.createElement('iframe');
      iframe.style.display = 'none';
      document.body.appendChild(iframe);
    }
    return iframe;
  };
})();
```

下次我们想创建`script`，我们又得把上边的代码复制一遍。

把管理单例的逻辑抽离出来：

```javascript
var getSingle = function(fn) {
  var result;
  return function() {
    return result || (result = fn.apply(this, arguments)); // 有result返回result，否则执行创建单例的函数
  };
};
```

调用：

```javascript
var createSingleScript = getSingle(function() {
  script = document.createElement('script');
  script.style.display = 'none';
  document.body.appendChild(script);
});
```

使用惰性单例（在需要请求数据的时候调用它）：

```javascript
document.getElementById('fetchDataBtn').onclick = function() {
  var singleScript = createSingleScript();
  singleScript.src = 'https://baidu.com';
};
```