# 策略模式
策略模式：要实现某一个功能，有多种方案可以选择。我们定义一系列的算法，把它们一个个封装起来，并且使它们可以相互转换。

## 一个例子
### 不好的写法

```javascript
var calculateBonus = function(performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return salary * 4;
  }
  if (performanceLevel === 'A') {
    return salary * 3;
  }
  if (performanceLevel === 'B') {
    return salary * 2;
  }
};
console.log(calculateBonus('B', 20000)); // 输出:40000 
console.log(calculateBonus('S', 6000)); // 输出:24000
```

### 使用组合函数重构代码

```javascript
var performanceS = function(salary) {
  return salary * 4;
};
var performanceA = function(salary) {
  return salary * 3;
};
var performanceB = function(salary) {
  return salary * 2;
};
var calculateBonus = function(performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return performanceS(salary);
  }
  if (performanceLevel === 'A') {
    return performanceA(salary);
  }
  if (performanceLevel === 'B') {
    return performanceB(salary);
  }
};

console.log(calculateBonus('A', 10000)); // 输出:30000
```

### 使用策略模式重构代码

传统的面向对象的策略模式：

```javascript
// 定义策略类
class performanceS {
  calculate(salary) {
    return salary * 4;
  }
}

class performanceA {
  calculate(salary) {
    return salary * 3;
  }
}

class performanceB {
  calculate(salary) {
    return salary * 2;
  }
}

// 定义奖金类
class Bonus {
  constructor() {
    this.salary = null; // 原始工资
    this.strategy = null; // 对应的策略对象
  }

  // 设定原始工资
  setSalary(salary) {
    this.salary = salary;
  }

  // 设置员工绩效等级对应的策略对象
  setStrategy(strategy) {
    this.strategy = strategy;
  }

  getBonus() {
    return this.strategy.calculate(this.salary);
  }
}

var bonus = new Bonus();
bonus.setSalary(10000);
bonus.setStrategy(new performanceS()); // 设置策略对象
console.log(bonus.getBonus()); // 输出:40000 
bonus.setStrategy(new performanceA()); // 设置策略对象
console.log(bonus.getBonus()); // 输出:30000
```

## JavaScript版本的策略模式
在`JavaScript`语言中，函数也是对象，所以更简单和直接的做法是把`strategy`直接定义为函数：

```javascript
var strategies = {
  S: function(salary) {
    return salary * 4;
  },
  A: function(salary) {
    return salary * 3;
  },
  B: function(salary) {
    return salary * 2;
  }
};

var calculateBonus = function(level, salary) {
  return strategies[level](salary);
};
console.log(calculateBonus('S', 20000)); // 输出:80000 
console.log(calculateBonus('A', 10000)); // 输出:30000
```

## 表单校验

```javascript
var registerForm = document.getElementById('registerForm');
registerForm.onsubmit = function() {
  if (registerForm.userName.value === '') {
    alert('用户名不能为空');
    return false;
  }
  if (registerForm.password.value.length < 6) {
    alert('密码长度不能少于 6 位');
    return false;
  }
  if (!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
    alert('手机号码格式不正确');
    return false;
  }
}
```

不好的点：

* `registerForm.onsubmit`函数比较庞大，包含了很多`if-else`语句，这些语句需要覆盖所有的校验规则。
* `registerForm.onsubmit`函数缺乏弹性，如果增加了一种新的校验规则，或者想把密码的长度校验从`6`改成`8`，我们都必须深入`registerForm.onsubmit`函数的内部实现，这是违反开放—封闭原则的。
* 算法的复用性差，如果在程序中增加了另外一个表单，这个表单也需要进行一些类似的校验，那我们很可能将这些校验逻辑复制得漫天遍野。


## 用策略模式重构表单校验

```javascript
var strategies = {
  isNonEmpty: function(value, errorMsg) {
    if (value === '') {
      return errorMsg;
    }
  },
  minLength: function(value, length, errorMsg) {
    if (value.length < length) {
      return errorMsg;
    }
  },
  isMobile: function(value, errorMsg) { // 手机号码格式
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg;
    }
  }
};

class Validator {
  constructor() {
    this.cache = []; // 保存校验规则
  }

  add(dom, rule, errorMsg) {
    var ary = rule.split(':'); // 把strategy和参数分开(参数在:之后)
    this.cache.push(function() { // 把校验的步骤用空函数包装起来，并且放入 cache
      var strategy = ary.shift(); // 用户挑选的 strategy
      ary.unshift(dom.value); // 把 input 的 value 添加进参数列表
      ary.push(errorMsg); // 把 errorMsg 添加进参数列表
      return strategies[strategy].apply(dom, ary);
    });
  }

  start() {
    for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
      var msg = validatorFunc(); // 开始校验，并取得校验后的返回信息 
      if (msg) { // 如果有确切的返回值，说明校验没有通过
        return msg;
      }
    }
  }
}

var validataFunc = function() {
  var validator = new Validator(); // 创建一个 validator 对象
  /***************添加一些校验规则****************/
  validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');
  validator.add(registerForm.password, 'minLength:6', '密码长度不能少于 6 位');
  validator.add(registerForm.phoneNumber, 'isMobile', '手机号码格式不正确');
  var errorMsg = validator.start(); // 获得校验结果
  return errorMsg; // 返回校验结果 
};

var registerForm = document.getElementById('registerForm');
registerForm.onsubmit = function() {
  var errorMsg = validataFunc(); // 如果 errorMsg 有确切的返回值，说明未通过校验
  if (errorMsg) {
    alert(errorMsg);
    return false; // 阻止表单提交 
  }
};
```

使用策略模式重构代码之后，我们仅仅通过“配置”的方式就可以完成一个表单的校验，这些校验规则也可以复用在程序的任何地方，还能作为插件的形式，方便地被移植到其他项目中。
在修改某个校验规则的时候，只需要编写或者改写少量的代码。比如我们想将用户名输入框的校验规则改成用户名不能少于 4 个字符。可以看到，这时候的修改是毫不费力的。代码如下:

把

```javascript
validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');
```

改成:

```javascript
validator.add(registerForm.userName, 'minLength:10', '用户名长度不能小于 10 位');
```

## 多种校验规则

```javascript
var strategies = {
  isNonEmpty: function(value, errorMsg) {
    if (value === '') {
      return errorMsg;
    }
  },
  minLength: function(value, length, errorMsg) {
    if (value.length < length) {
      return errorMsg;
    }
  },
  isMobile: function(value, errorMsg) { // 手机号码格式
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg;
    }
  }
};

class Validator {
  constructor() {
    this.cache = []; // 保存校验规则
  }

  add(dom, rules) {
    var self = this;
    for (var i = 0, rule; rule = rules[i++];) {
      (function(rule) {
        var strategyAry = rule.strategy.split(':');
        var errorMsg = rule.errorMsg;
        self.cache.push(function() {
          var strategy = strategyAry.shift();
          strategyAry.unshift(dom.value);
          strategyAry.push(errorMsg);
          return strategies[strategy].apply(dom, strategyAry);
        });
      })(rule);
    }
  }

  start() {
    for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
      var errorMsg = validatorFunc();
      if (errorMsg) {
        return errorMsg;
      }
    }
  }
}

var registerForm = document.getElementById('registerForm');

var validataFunc = function() {
  var validator = new Validator();
  validator.add(registerForm.userName, [{
    strategy: 'isNonEmpty',
    errorMsg: '用户名不能为空'
  }, {
    strategy: 'minLength:6',
    errorMsg: '用户名长度不能小于 10 位'
  }]);
  validator.add(registerForm.password, [{
    strategy: 'minLength:6',
    errorMsg: '密码长度不能小于 6 位'
  }]);
  validator.add(registerForm.phoneNumber, [{
    strategy: 'isMobile',
    errorMsg: '手机号码格式不正确'
  }]);
  var errorMsg = validator.start();
  return errorMsg;
};

registerForm.onsubmit = function() {
  var errorMsg = validataFunc();
  if (errorMsg) {
    alert(errorMsg);
    return false;
  }
};
```


## 很久之前，邮件里提到的性能问题：

```javascript
const map = ['A', 'B', 'C', 'D', 'E', 'F'];

const start = new Date().getTime();

var data;

for (var i = 0; i < 66666666; i++) {
  data = map[i % 6];
}

const end = new Date().getTime();

console.log(end - start);
```

```javascript
const map = ['A', 'B', 'C', 'D', 'E', 'F'];

const start = new Date().getTime();

const testSwitch = (key) => {
  switch(key) {
    case 1 :
      return 'A';
    case 2 :
      return 'B';
    case 3 :
      return 'C';
    case 4 :
      return 'D';
    case 5 :
      return 'E';
    case 6 :
      return 'F';
  }
}

var data;

for (var i = 0; i < 66666666; i++) {
  data = testSwitch(i);
}

const end = new Date().getTime();

console.log(end - start);
```

然而这点性能问题好像并不在意什么，我还是喜欢用`策略模式`。

## 优缺点

* 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。
* 策略模式提供了对开放—封闭原则的完美支持，将算法封装在独立的`strategy`中，使得它们易于切换，易于理解，易于扩展。
* 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。
* 在策略模式中利用组合和委托来让`Context`拥有执行算法的能力，这也是继承的一种更轻便的替代方案。


当然，策略模式也有一些缺点，但这些缺点并不严重。

首先，使用策略模式会在程序中增加许多策略类或者策略对象，但实际上这比把它们负责的逻辑堆砌在`Context`中要好。
其次，要使用策略模式，必须了解所有的`strategy`（策略的名字要通过API暴露），必须了解各个`strategy`之间的不同点，这样才能选择一个合适的`strategy`。比如，我们要选择一种合适的旅游出行路线，必须先了解选 择飞机、火车、自行车等方案的细节。此时`strategy`要向客户暴露它的所有实现，这是违反最少知识原则的。