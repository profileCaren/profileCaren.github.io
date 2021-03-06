---
title: 策略模式- JavaScript 设计模式(一)
date: 2018-10-25 09:56:08
tags: [JavaScript设计模式, 读书笔记]
abstract: 策略模式：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。本文通过两个例子讲述 JavaScript 中的策略模式。
categories: 
- [JavaScript 设计模式]
---

# 写在“设计模式系列“第一篇的开头

本系列作为《JavaScript 设计模式与开发实践》一书的读书笔记，将介绍前端开发中常用的一些设计模式。目前已经有非常多的优秀文章将设计模式讲的很好了，笔者写作的初衷有以下几点：

1. 备忘。
2. [教是最好的学](http://mindhacks.cn/2009/02/15/why-you-should-start-blogging-now/)。只有当我们能将知识准确、完整的用自己的语言阐述出来，才能说明我真的懂了。

总体而言，是为了巩固知识。
本系列所有文章的讲述逻辑一般将会是这样的：

1. 先讲述从一个具体的需求。
2. 讲述正常思维的实现以及其缺点，缺点一般是代码难以维护。
3. 讲述如何使用某个设计模式实现该需求及其优点。
4. 该设计模式的拓展。

之所以使用这样的介绍逻辑，是因为我们就是这样思考的。当我们遇到一个业务需求，心里面并不是想着：

> 好的，这次我要用命令模式实现它！

而是在遇到一个问题的时候，突然感觉这个需求好像和某个设计模式的适用场景情境很像，于是心想

> 咦，这个场景好像和命令模式的场景很相似，我去看看能不能用命令模式实现它。

所以，**我们的介绍重点不在于设计模式，而在于场景**。读者需要理解的重点也是为何这个场景需要使用某个模式，优点体现在哪里，而不是这个设计模式的定义是什么。

话不多说，让我们开始介绍策略模式。按照惯例，应该先介绍单例模式，但单例模式太简单了，笔者不愿赘述，故直接从策略模式开始。

# 例子1：计算年终奖

现在假设一个公司的年终奖是根据**工资基数**和**绩效等级**来发放的：

| 绩效等级 | 年终奖数额 |
| -------- | ---------- |
| S        | 四倍工资   |
| A        | 三倍工资   |
| B        | 两倍工资   |

## 初级工程师版本

一个新手的JavaScript工程师会这样写:

```js
const calculateBonus = function(performanceLevel, salary) {
  if (performanceLevel === "S") {
    return salary * 4;
  }
  if (performanceLevel === "A") {
    return salary * 3;
  }
  if (performanceLevel === "B") {
    return salary * 2;
  }
};
```

这带来的问题是假设你要新增一个`C`绩效等级，就要去增加一个`if`，新增这个`if`的同时又有可能改动到别的逻辑。一个重要的原则是，凡是你发现你代码里面很多`if`判断，说明它可以被优化。具体的优化方式有很多种，可以点击这个[链接](https://jrsinclair.com/articles/2017/javascript-but-less-iffy/)去了解一下。

## 中级工程师版本(策略模式)

一个略有经验的工程师则会这样写：
```js
const Strategies = {
  "S": salary=> salary * 4,
  "A": salary=> salary * 3,
  "B": salary=> salary * 2,
}

const calculateBonus = (performanceLevel, salary) =>{
  return Strategies[performanceLevel](salary);
}

// 一个绩效等级为S，薪资为15000的员工。
calculateBonus('S', 15000); // 60000

```

实际上, 这就是策略模式。
![](https://user-gold-cdn.xitu.io/2018/10/25/166a91c67a20940a?w=250&h=250&f=png&s=31371)

在传统面向对象语言中，要实现策略模式需要定义
1. 一组策略类，策略类封装了具体的算法。
2. 环境类 Context，Context 接受客户的请求，随后将请求委托给某一个策略类。

太麻烦了。而 JavaScript 则由于它丰富的表达能力，可以将`calculateBonus`函数看成是Context，将`Strategies`对象看成一组策略，轻松地实现了一个策略模式。

## 额外的tips

上文“中级工程师”版本的写法其实还有个小问题，就是 performanceLevel 这个变量是硬编码的，比如`"S"`, `"A"`, `"B"`。这么短的一个字符可能不会带来什么问题，但假设这个策略的名称比较长，比如`"Christina"`，那你就很容易敲错，敲错就出 bug 了嘛。

我们可以用ES6的[Symbol](http://es6.ruanyifeng.com/#docs/symbol)和[解构](http://es6.ruanyifeng.com/#docs/destructuring)把它改写一下：

```js
const PERFORMANCE_LEVEL = {S:'S', A: 'A', B: 'B'};
const {S, A, B}  = PERFORMANCE_LEVEL;
const Strategies = {
  [S]: salary=> salary * 4,
  [A]: salary=> salary * 3,
  [B]: salary=> salary * 2,
}
const calculateBonus = (performanceLevel, salary) =>{
  return Strategies[performanceLevel](salary);
}

calculateBonus(S, 15000); // 60000
```

这样会更好一点点，但当然，这和设计模式关系不大。
这个例子较为简单，接下来我们讲述一个更为常见的表单校验的例子。

# 例子2：表单校验

表单几乎是前端工程师写的最多的东西，登录注册、修改用户信息都离不开表单。在提交一个表单给后台之前，客户端一般要对一些数据进行校验，比如 Email、电话是否符合格式，某些必填字段是否为空等等。

现在假设我们在编写一个注册页面，校验规则如下：
- 用户名不能为空
- 密码长度不能少于6位
- 手机号码必须符合格式。

## 常规思维写法

```html
<form action="" id="registerForm" method="post">
  请输入用户名: <input type="text" name="userName"><br>
  请输入密码: <input type="password" name="password"><br>
  请输入手机号码: <input type="phone" name="phoneNumber"><br>
  
  <input type="submit">
</form>
```

```js
let registerForm = document.getElementById('registerForm');

registerForm.onsubmit = function(){
  let {userName, password, phoneNumber} = registerForm;
  if(userName.value === '' ){
    alert('用户名不能为空');
    return false;
  }
  if(password.value.length < 6){
    alert('密码长度不能少于6位');
    return false;
  }
  if(!/(^1[3|5|8][0-9]{9}$)/.test(phoneNumber.value)){
    alert('手机号码格式不正确');
    return false;
  }
}
```
进入这个[Codepen](https://codepen.io/caren11/pen/mzQaWm)玩一下。

可以看到，在`onsubmit`函数中，我们对`registerForm`的每个字段逐一验证，这是十分符合逻辑的写法。但是略有经验的工程师会知道，将来我们可能要新增一些字段，比如邮箱；还有可能要对某些字段新增校验规则，比如要求密码必须同时包含大小写字符和数字。这样一来，我们就要经常对`onsubmit`函数进行修改。

## 策略模式写法

原书的作者曾探大佬重构这段代码的时候，上来就封装了一个策略对象，笔者认为这样不太好理解。我们来先从目的出发，反推我们应该如何使用策略模式进行重构。一些阅读代码能力很强的读者可以直接跳到尾部的完整代码直接进行查看。

### 思路

读者可以思考一分钟，你会如何重构这部分代码。

笔者的思路是这样的。基于我们写表单的经验，我们知道在将来我们可能会

1. 新增某些字段。
2. 对某个字段进行多种不同的校验，输出不同的提示信息。
3. 校验别的表单，但我不希望重写一份校验规则。

基于以上假设，我们想要的效果是这样的：

a. 在表单提交的时候，只需要调用 `validate()` 函数即可进行校验:
```js
registerForm.onsubmit = () => {
  let errorMessage = validate();
  if (errprMessage) {
    alert(errorMessage);
    return false;
  }
  ...
};
```

b. `validate()`函数负责创建一个`Validator`，添加相应的规则，并执行校验：
```js
let validate = () => {
  let validator = new Validator();
  // 一次添加单个规则, (longerThan6 和 isPhone 都是函数)
  validator.add(registerForm.password, longerThan6, "密码长度必须大于6位");
  validator.add(registerForm.phoneNumber, isPhone, "电话号码格式错误");

  // 一次添加多个规则
  validator.add(registerForm.userName, [
    {
      rule: val => val.length > 6,
      message: "用户名长度必须大于六位"
    },
    { rule: notEmpty, message: "用户名不能为空" }
  ]);

  // 开始校验，成功则返回false，失败则返回对应的errorMessage。
  return validator.start();
};
```

目标就是这样，那么关键就是`Validator`这个类的实现了。

### Validator类的实现

我们刚才看到，`Validator`类主要包含两个方法，`add`和`start`，除此之外，我们还需要一个`rules`数组，用于缓存待验证的值、验证函数、以及 error message. 具体代码如下：

```js
class Validator {
  constructor() {
    this.rules = [];
  }

  /**
   * 该方法负责添加验证规则，存储在 this.rules 中。
   * 其接受三个参数，第一个参数是input元素( Element )
   * 第二个参数可能是数组或函数，若第二个参数是数组，忽略第三个参数，
   * 且该数组的元素结构必须是 { validateFunc: Function, errorMessage: String }。
   *
   * @param {Element} inputElement
   * @param {Array | Function} rules
   * @param {String} message
   * @memberof Validator
   */

  add(inputElement, rules, errorMessage) {
    let val = inputElement.value; 

    // 如果第二个参数是函数，那就是用于验证的函数咯。
    if (typeof rules === "function") {
      let validateFunc = rules; // 没啥意义的赋值，为了你更好的理解。

      this.rules.push({
        value: val,
        validateFunc: validateFunc,
        errorMessage: errorMessage
      });
      return;
    }

    // 如果第二个参数是数组，那就是一组规则，忽略第三个参数。
    if (Array.isArray(rules)) {
      for (let rule of rules) {
        this.rules.push({
          value: val,
          validateFunc: rule.validateFunc,
          errorMessage: rule.errorMessage
        });
      }
      return;
    }
  }

  start() {
    for (let rule of this.rules) {
      let { value, validateFunc, errorMessage } = rule;
      if (!validateFunc(value)) {
        return errorMessage;
      }
    }
    // 验证通过。
    return false;
  }
}

```

### 最后还需要一个规则对象(策略对象)

细心的读者会发现，在`validate`函数中用了三个变量,`longerThan6`, `isPhone`和`notEmpty`。他们其实都是函数，用于被调用来验证值的正确性嘛。

为了可复用，我们要将一些常见的验证规则保存起来：

```js
const Rules = {
  longerThan6: val => val && val.length >= 6,
  isPhone: val => val && /(^1[3|5|8][0-9]{9}$)/.test(val),
  notEmpty: val => val && val.length != 0
};
```

要使用的时候:
```js
let {longerThan6, isPhone, notEmpty} = Rules;
```

解构语法真是好用啊我的老天鹅。

### 完整代码

[Codepen](https://codepen.io/caren11/pen/rqQbZB?editors=1111)


# 解析策略模式

策略模式的定义是：**定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。**

前文提及，在传统面向对象语言中，一个基于策略模式的程序至少由两部分组成:
1. **一组策略类**: 策略类封装了具体的算法，并负责具体的计算过程。
2. **一个环境类** Context: Context 接受用户的请求，随后把请求委托给一个策略类。

而在美丽的 JavaScript 中，一组策略类完全可以使用一个**策略对象**代替。这个策略对象的 key 是各个策略名称， value 是对应策略的具体算法。

如第一个计算工资例子中的:
```js
const Strategies = {
  "S": salary=> salary * 4,
  "A": salary=> salary * 3,
  "B": salary=> salary * 2,
}
```

以及第二个表单校验例子中的:
```js
const Rules = {****
  longerThan6: val => val && val.length >= 6,
  isPhone: val => val && /(^1[3|5|8][0-9]{9}$)/.test(val),
  notEmpty: val => val && val.length != 0
};

```
那么**环境类**呢？其实在 JavaScript 中，随便写一个函数就可以了。

简而言之，我靠..写到最后发现 JavaScript 中根本没有策略模式。基本上只要你熟悉了通过一个对象的方式来避免某些情况下的多个`if`，你就算是掌握了策略模式了。既然如此，我就不赘述策略模式的优点了，大体是避免了多个`if`、方便策略函数的重用等等。

顺便看了一下原书，原来作者在末尾也说了这一点:

> 在函数作为一等对象的语言中，策略模式是隐形的。strategy就是值作为函数的变量。
