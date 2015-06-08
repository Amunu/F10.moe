title: erlang创建模块与变量定义
date: 2014-10-17 13:39:25
tags: [erlang, 学习笔记]
---

### 基本步骤

想要自行开发实际的 Erlang 程序， 就必须把编写的代码放置到一个或者多个模块中。要新开发一个可供自己和其他人重用的模块，步骤如下：

- 编写源码文件
- 编译
- 加载已编译的模块，或将它放到加载路径中以便自动加载

### my_module.erl

首先新建一个 *my_module.erl* 文件：

```
%% A simple erlang module
-module(my_module).
-export([pie/0]).
pie() -> 
	3.14.

```

- ``pie() -> 3.14.`` 称为 *函数定义* 。*函数首部（函数名和参数列表）* 和 *函数体（函数的用途）* 由箭头 **->** 隔离。

- 函数不需要 ``return`` 之类的关键字：函数的返回值就是函数体中表达式的值。

- 注意函数定义的末尾必须加上句号（``.``）。这里的（``.``）可以看做其他语言中熟悉的（``;``） 。

- Erlang 的注释, 用 ``%`` 表示， 根据规范与代码在一行的注释以 ``%`` 开头，独占一行的注释以 ``%%`` 开头。

- -module(my_module). 除了注释代码的第一行一定是模块声明。总的来说，Erlang 中既非函数也非注释的东西都属于声明。 模块声明是不可或缺的， 且它指定的名字必须与文件名相符。

- -export([...]). 是 *导出声明* ，它会告知编译器哪些函数是 *外部可见* 的。此处没有列出的函数都是模块的内部函数。上面那个例子中只有一个函数，要想调用它就得把它放入 *导入列表*。 只能同时给定函数名和元数(此处为0)才能唯一确定一个函数， 因此要写成``pie/0``。

### 编译和加载

编译模块时会产生一个和模块名对应的拓展名为``.beam``的文件，其中包含着可被Erlang系统加载执行的指令。

- 在 shell 编译： 最简单的方法就是调用 shell 函数 ``c(...)``, 它不光负责模块编译还能完成模块加载，能即刻进行测试。
- 在之前的文件夹下启动Erlang(输入erl并回车)执行：
```
1> c(my_module).
{ok,my_module}
2> my_module:pie().
3.14
```

- 输出了{ok,my_module}， 代表成功了。此时可以看到文件夹下又多了个 my_module.beam 文件，这就是已编译版本的模块，也称``目标文件``。有了这个文件后就直接可以调用之前的模块了，由于已经编译成功。
```
%% 退出erlang后再次进入不用再次编译
1> my_module:pie().
3.14
```

- 调用``code:get_path().``可以检查当前代码路径设置。
```
3> code:get_path().
[".","/usr/lib/erlang/lib/kernel-3.0.1/ebin",
 "/usr/lib/erlang/lib/stdlib-2.1/ebin",
 "/usr/lib/erlang/lib/xmerl-1.3.7/ebin",
 "/usr/lib/erlang/lib/wx-1.3/ebin",
 "/usr/lib/erlang/lib/webtool-0.8.10/ebin",
 "/usr/lib/erlang/lib/typer-0.9.8/ebin",
 "/usr/lib/erlang/lib/tools-2.6.15/ebin",
 "/usr/lib/erlang/lib/test_server-3.7.1/ebin",
 "/usr/lib/erlang/lib/syntax_tools-1.6.15/ebin",
 "/usr/lib/erlang/lib/ssl-5.3.5/ebin",
 "/usr/lib/erlang/lib/ssh-3.0.3/ebin",
 "/usr/lib/erlang/lib/snmp-4.25.1/ebin",
 "/usr/lib/erlang/lib/sasl-2.4/ebin",
 "/usr/lib/erlang/lib/runtime_tools-1.8.14/ebin",
 "/usr/lib/erlang/lib/reltool-0.6.6/ebin",
 "/usr/lib/erlang/lib/public_key-0.22/ebin",
 "/usr/lib/erlang/lib/percept-0.8.9/ebin",
 "/usr/lib/erlang/lib/parsetools-2.0.11/ebin",
 "/usr/lib/erlang/lib/otp_mibs-1.0.9/ebin",
 "/usr/lib/erlang/lib/os_mon-2.2.15/ebin",
 "/usr/lib/erlang/lib/orber-3.6.27/ebin",
 "/usr/lib/erlang/lib/odbc-2.10.20/ebin",
 "/usr/lib/erlang/lib/observer-2.0.1/ebin",
 "/usr/lib/erlang/lib/mnesia-4.12.1/ebin",
 "/usr/lib/erlang/lib/megaco-3.17.1/ebin",
 "/usr/lib/erlang/lib/jinterface-1.5.9",
 "/usr/lib/erlang/lib/inets-5.10.2/ebin",
 [...]|...]

```

### 变量

Erlang 最显著的特点就是在于``变量名必须以大写字母开头``！由于小写字母的名字已经被用于[原子](http://f10.moe/2014/10/16/erlang-%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/)了。
- 变量名中的单词以 **驼峰体（CamelCase）** 隔开， 这是 Erlang 变量的标准命名风格。例如：Z, Name, ShortName
- 变量名也可以用 **下划线** 开头。一般情况下，按常规第二个字符通常为大写字母。例如：_Z, _Name, _Short_Name
- ``Erlang 的变量被严格地限定只能接受**单次赋值**`` ， 也就是说，变量一旦被赋值，即变量被绑定到某个值上， 该变量在其整个作用域内一直持有这个值。

### 在shell中使用变量

下面都是只是在 **shell** 中变量作用域的工作方式。在 **模块** 内，作用域依赖于函数定义之类的东西，在作用域之前是``无法提前遗忘变量绑定``的。

```
1> X = 1.
1
2> X.
1
3> X + 4.
5

```
- 从上面的结果可以看到，X仍在shell的作用域中。可以调用``f()``函数来让 shell 遗忘先前绑定的所有变量：

---

```
4> f().
ok
5> X.
* 1: variable 'X' is unbound
6> 

```

- 一旦被遗忘，那就可以继续赋值了：

---

```
6> X = 100.
100
7> X = 99.
** exception error: no match of right hand side value 99
8> f().
ok
9> X = 99.
99

```

- 当 X 被赋值后，等号(=)的作用就从赋值变成了 **完全相等运算符**

在某些场合，你会遇到大量基本相同又有些差距的数据，从而不得不使用大量的变量。这种时候就应该尝试将代码切分成为独立的函数。令每个函数都有自己的X，并专心解决整个问题中的单个分解步骤。

### 总结

学到这，还是被 Erlang 的变量的设定吓了下（没错！人家就是这么胆小！）它更像是解方程的时候的X变量，从头到尾都不会变。但据说这样会对程序的稳定性、可拓展性有着深远的影响0. 0 具体怎么深远呢？！等我继续学下去...



