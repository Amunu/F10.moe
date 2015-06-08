title: ES6 Generator 基础
date: 2014-10-27 12:04:57
tags: [node, koa, ES6 Generator]
---

为了更好的回避js的回调金字塔，就可以使用ES6 generator 来取代回调函数。

### installation

node.js 中需要 **0.11.***版本， 由于我装了[**nvm**](http://f10.moe/2014/10/27/nvm%E7%AE%A1%E7%90%86Node%E7%89%88%E6%9C%AC/)， 所以直接可以用 ``nvm install 0.11.13`` 来安装 **node** 版本。

同时启动 **harmony generator** 还需要使用 ``node -harmony`` 。

在 **chrome** 中使用该特效需要打开 chrome://flags/， 然后将 **harmony** 设置为 **enable** , 重启 **chrome** 。

### generator

那么什么是 generator 呢。其实，它是一个函数，但是这个函数的行为比较特殊：

- 它不直接执行逻辑，而是用来生成一个对象。正如它的名字一样，他是一个生成器。

- 它所生成的对象中的函数可以把逻辑拆开来， 一段一段调用执行， 而不像普通函数，从头执行到尾， 一次执行完毕。

- 你可以暂停它的执行。当你请求它时，它会给出对应的响应，但是不会继续往下执行，直到你再次请求。

generator 的语法：

- 函数声明/函数表达式 的关键字 ``function`` 后多了个 ``*`` 。

- 函数体中多了 ``yield`` 运算符。yield 有点像是 return 关键字，因为它们都返回一个值，但是这里的函数会在 yield 之后。

例如：

```
➜  test  node -harmony       
> function * GenA(){
>     console.log('from GenA, first.');
>     yield 1;
>     console.log('from GenA, second.');
>     var value3 = yield 2;
>     console.log('from GenA, third.',value3);
>     return 3;
> }
undefined
> var a = new GenA();
undefined
> a.next();
from GenA, first.
{ value: 1, done: false }
> a.next();
from GenA, second.
{ value: 2, done: false }
> a.next(2333);
from GenA, third. 2333
{ value: 3, done: true }
> a.next();
{ value: undefined, done: true }

```

例子解释：

- 在调用 ``GenA()`` 时，函数体中的逻辑并不会执行（控制台没有输出），直接调用 ``a.next()`` 时才会执行

-  ``a`` 是一个对象，它由生成器 ``GenA()`` 实例化而来（事实上，不需要new运算符也是一样的结果）

- 调用 ``a.next()`` 时，函数体中的逻辑才开始真正执行，每次调用时会到 ``yield`` 语句结束，并将 ``yield`` 的运算数作为结果返回

- ``a.next()`` 返回的结果是一个对象，对 ``yield`` 的运算数做了包装，并带上了 ``done`` 属性

- 当 ``done`` 属性为 ``false`` 时，表示该函数逻辑还未执行完，可以调用 ``a.next()`` 继续执行，否则不可继续调用

- 最后一次返回的结果为 ``return`` 语句返回的结果，且 ``done`` 值为 ``true`` 。如果不写 ``return`` ，则值为 ``undefined``

- ``value3 = yield 2`` 这句是指，这一段逻辑返回2，在下一次调用 ``a.next()`` 时，将参数赋给 ``value3`` 。换句话说，这句只执行了后面半段就暂停了，等到再次调用 ``a.next()`` 时才会将参数赋给value3并继续执行下面的逻辑


### 代替回调金字塔

首先先写一个耗时的异步的函数：

```
function delay (time, callback) {
	setTimeout(function(){ callback && callback()}, time);
}

```

然后构造金字塔：

```
delay(200, function(){
	delay(300, function(){
		delay(400, function(){
			delay(500, function(){
				delay(100, function(){
					console.log('finish');
				});
			});
		});
	});
});

```

接下来我们来定义一个 generator 函数：


```
function* doDelay(){
	delay(200, function(){console.log('1');});
	delay(300, function(){console.log('1');});
	delay(400, function(){console.log('1');});
	delay(500, function(){console.log('1');});
	delay(600, function(){console.log('1');});
}
var a = new doDelay();
a.next();
```
此时函数依旧是异步的，由于我们当前还未放入任何的yield语句。
所以执行的时候会直接输出5个1。
所以改成下面这样的：

```
function* doDelay(){
	yield delay(200, function(){console.log('1');});
	yield delay(300, function(){console.log('1');});
	yield delay(400, function(){console.log('1');});
	yield delay(500, function(){console.log('1');});
	yield delay(600, function(){console.log('1');});
}
var a = new doDelay();
a.next();

```

---

现在我们要编写一个 ``resume`` 函数来推动 generator。

如果你看看其他使用generator代替回调函数的例子，你会看到generator函数总是被另一个函数包裹着 – 通常是一个叫做“run”或者“execute”的函数。这样做的目的如下：

- 接收一个 ``generator`` 作为参数

- 使用这个 ``generator`` 来创建一个新的 ``generator`` 迭代器对象，我们将调用它的 ``next`` 方法。

- 创建一个 ``resume`` 函数来使用这个 ``generator`` 迭代器对象来推进 ``generator``

- 将 ``resume`` 函数传递给这个 ``generator`` 以便 ``generator`` 能够访问 ``resume``

- 在最开始时调用 ``next()`` 函数，以便我们的代码在碰到第一个 ``yield`` 之前开始执行

---

创建 ``run()`` 函数：

```
function run(generatorFunc){
	var generatorltr = generatorFunc(resume);
	function resume(callbackValue){
		generatorltr.next(callbackValue);
	}
	generatorltr.next();
}
```

现在我们有了一个能够接收一个 ``generator`` 函数的函数，并为它传递了一个了解如何推进 ``generator`` 迭代器对象的函数。

注意到我们在 ``resume`` 函数中用到了 ``next`` 的第二个特性。

``resume`` 是被传递给 ``delay`` 的回调函数，因此它接收 ``delay`` 函数提供的值。``resume`` 将这个值传递给 ``next`` ，因此 ``yield`` 语句的结果实际上是我们异步函数的结果。

接下来将之前的 ``generator`` 函数包上 ``run``

```
function delay (time, callback) {
	setTimeout(function(){ callback && callback(1)}, time);
}

function run(generatorFunc){
	var generatorltr = generatorFunc(resume);
	function resume(callbackValue){
		generatorltr.next(callbackValue);
	}
	generatorltr.next();
}

run(function* doDelay(resume){
	console.log(yield delay(200, resume));
	console.log(yield delay(300, resume));
	console.log(yield delay(400, resume));
	console.log(yield delay(500, resume));
	console.log(yield delay(600, resume));
});

```

- run接收了我们的generator并创建了一个resume函数
- run创建了一个generator迭代器对象（我们在它上面调用next方法），提供了resume函数。接着它推动了generator迭代器向前运行。
- 我们的generator碰到了第一个yield语句并且调用delay。接着这个generator暂停。
- delay在200ms之后完成然后调用resume。
- resume告诉我们的generator进行下一步。它将结果传递给delay以便console能够将它打印出来。
- 我们的generator碰到了第二个yield，调用delay然后再次暂停。 
- delay等待300ms之后调用resume回调函数。
- resume再次推进generator。
- 重复继续

理解了 run 的步骤，tj大神的 [co](https://github.com/tj/co) 框架也差不多能起来了。

---
### 参考资料：
 + http://www.toobug.net/article/learning_es6_generator.html
 + http://bg.biedalian.com/2013/12/21/harmony-generator.html
 + http://www.html-js.com/article/A-day-to-learn-Javascript-to-replace-the-callback-function-with-ES6-Generator
