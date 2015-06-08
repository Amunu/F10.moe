title: What is "this"?
date: 2015-01-26 14:14:46
tags: [node, 学习笔记]
---

### 作用域和闭包

相信接触过编程的人，大多都是知道作用域的，像下面两个关于全局变量和本地变量的例子：

```
//定义全局变量
var name = "Minary";
var age = 21;
console.log("Hello " + name + ".  Wow, you are " + age + " years old.");
```
上面输出： Hello Minary.  Wow, you are 21 years old.

---
```
//定义本地变量
function newScope() {
  var name = "Minary";
  var age = 21;
}
console.log("Hello " + name + ".  Wow, you are " + age + " years old.");
```
上面的例子会报 name is not defined 的错误。因为尝试从全局范围中获取本地变量。

---
**作用域是闭包实现的关键**，下面是 wiki 对闭包和作用域的解释：

> In computer science, a closure is a first-class function with free variables that are bound in the lexical environment. Such a function is said to be "closed over" its free variables. A closure is defined within the scope of its free variables, and the extent of those variables is at least as long as the lifetime of the closure itself.

什么意思呢？ 可以看下面的例子：

```
var name = "outer";
function one() {
  var name = "middle";
  var other = "findme";
  function two() {
    var name = "inner";
    // 这里 `name` 为 "inner" ，`other` 为 "findme"
    console.dir({name: name, other: other});
  }
  two();
  // 这里 `name` 为 "middle" ，`other` 为 "findme"
  console.dir({name: name, other: other});
}
one();
// 这里 `name` 为 "outer" ，`other` 为 undefined.
console.dir({name: name});
console.dir({other: other});
```
也就是说本地作用域可以访问全局变量， 但是全局作用域不可以访问本地变量。 本地变量可以重新定义一个和全局变量一样名字的变量， 这时候再在本地作用域内访问像上面 `name` 一样的变量时，全局的 `name` 被本地覆盖。 但是出了这个本地作用域， 全局变量仍未改变。

---
那么要怎么访问本地作用域里的变量呢？看下面这个例子：

```
// 新建一个函数并 return 一个闭包函数
function myModule() {
  var name = "Minary", age = 21;
  return function greet() {
    return "Hello " + name + ".  Wow, you are " + age + " years old.";
  }
}
// call `myModule` to get a closure out of it.
var greeter = myModule();
// Call the closure
greeter();
```

`name` 和 `age` 变量是 `MyModule` 函数里的本地变量， 但是我们在全局作用域里通过执行 `greeter` 来访问，就不会报错。
这是因为 `greet` 函数的作用域里本来就有 `name` 和 `age` 这两个变量，所以直接访问不会报错。
基本上变量都是从它的作用域里获取所请求的变量。

### this

除了作用域，JavaScript 增加了另一层特殊的作用域，通过特殊的关键字 `this` 来实现。这个关键字的用法除了不能被修改，其他看起来和普通的变量一样。
它作为一个对象，你可以通过正常点或括号，得到它的属性。神奇的是， `this` 的值取决于调用它的情况。一般情况下，它的值就是接收到的信息。例如：

```
var Person = {
  name: "Minary",
  age: 21,
  greeting: function () {
    return "Hello " + this.name + ".  Wow, you are " + this.age + " years old.";
  }
};

Person.greeting(); //'Hello Minary.  Wow, you are 21 years old.'

```

上面的代码看起来 `this` 几乎像其他语言中的对象, 这说明被他骗了。作为这个 `Person` 对象的创造者， 你没有把握 `this` 会和 `Person` 一样， 比方说, 想在把 `greeting` 函数把保存在其他地方：

```
var greeting = Person.greeting;
greeting(); // 'Hello undefined.  Wow, you are undefined years old.'
```

`this.name` 和 `this.age` 就会为 `undefined`， 这是因为在 `greeting` 函数现在在全局对象里， 而不是 `Person` 对象里。 因此这里的 `this.name` 和 `this.age` 是在全局范围里找, 所以是找不到的。

所以下面这个例子就可以了：

```
var Dog = {
  name: "Dog",
  age: 2,
  greeting: Person.greeting
}

Dog.greeting(); // 'Hello Dog.  Wow, you are 2 years old.'
```
因为这时候 this 会在 `Dog` 作用域里找。

### 驯服 this

先看下面的例子：

```
var Alien = {
  name: "Zygoff",
  age: 5432
}

Person.greeting.call(Alien); // 'Hello Zygoff.  Wow, you are 5432 years old.'
```
对比一下之前的例子， 上面这个 `Alien` 对象里没有半句关于 `greeting` 函数， 但是我们仍旧可以使用它。
这是因为我们 `call` 了 `Person.greeting` 函数， 但是却把 `Alicn` 的值作为 `this` 注入。
我们还可以用 `apply` ，用法和上面的例子类似在没有额外的参数下。

---
让我们写个类函数可以被其他包含有 `name` 和 `age` 的对象使用：

```
function makeOlder(years, newname) {
  this.age += years;
  if (newname) {
    this.name = newname;
  }
}
```
我们可以通过 call 或者 apply 来调用它：

```
makeOlder.call(Person, 2, "Old Tim");
makeOlder.apply(Dog, [1, "Shaggy"]);
console.dir({Person: Person, Dog: Dog}); //{ Person: { name: 'Old Tim', age: 30, greeting: [Function] },Dog: { name: 'Shaggy', age: 111, greeting: [Function] } }

```

### 绑定 "this"

有时候我们更喜欢OPP风格的代码并想让JS也这样做。 我们并不喜欢 `this` 的改变取决于调用的时间。下面一个jQuery的简单例子：

```
Cart = {
  items: [1,4,2],
  onClick: function () {
    // Do something with this.items.
  }
}
$("#mybutton").click(Cart.onClick);
```
虽然上面的看起来不错，其实有个大坑等着你。 尽管已经有了 `Cart.onClick` ，我们先不调用它。 jQuery代码将接受的是一些参数，在这一点上它没有办法知道 `onClick` 是来自 `Cart` 的对象。你的 `this` 最后并不会像你期望的那样被called。

结合之前的闭包和作用域的知识让这个 `this` 就像它在大多数面向对象的语言那样。

```
$("#mybutton").click(function () { Cart.onClick() });
```

我们创建了一个闭包然后调用 ` Cart.onClick()` ， 现在没有任何的参数和返回值， 可以改成下面这样：

```
$("#mybutton").click(function () { return Cart.onClick.apply(Cart, arguments) });
```

现在已经成功了，但是如果你不知道 `arguments` 是另一个关键字（一个类似数组的对象包含了当前最内部函数的参数
）代码就会变得难度难懂。

如果 `Cart` 是个全局访问的单个对象，我们可以只使用变量 `Cart` 而不是依靠 `this` 。 但往往并不会这样，当你有“类”的对象共享的功能。

可以按下面那样修改 `Cart.onClick` 来使这个 `this` 一直在 `Cart` 里。

```
function bind(fn, scope) {
  return function () {
    return fn.apply(scope, arguments);
  }
}
Cart.onClick = bind(Cart.onClick, Cart);
$("#mybutton").click(Cart.onClick);
```

这里有很多种方式都能实现这样的效果，事实上他不是最好的解决方法。在上面，我们创建了一个闭包并把范围嵌在里面， 然后我们通过绑定闭包和使用 `apply` 自动传入参数并且自动返回值来替换 `Cart.onClick` 。

### link

http://howtonode.org/what-is-this
