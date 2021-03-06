title: erlang模式匹配
date: 2014-10-20 11:21:47
tags: [erlang, 学习笔记]
---

模式匹配是Erlang不可或缺的功能。它的重要作用：

- 选定控制流分支
- 完成变量赋值（绑定）
- 拆解数据结构（选择和提取各个组成部分）

### = 运算符就是模式匹配

我们将 = 称为匹配运算符，这是因为它的功能就是 *模式匹配* ，而不是赋值。运算符的左侧是一个 *模式* , 右侧是一个普通表达式。
做模式匹配时，首先计算右侧的表达式，得到一个值。然后拿着该值去匹配左侧的模式。比如 17 = 42 或者 true = flase， 则匹配宣告失败并抛出一个原因代码（reason code）为badmatch的异常。

若左侧为单个变量： X = 17，就意味着17 和 X 绑定。

见如下代码：

```
1> {A, B, C} = {1, 2, 3}.
{1,2,3}
2> A.
1
3> B.
2
4> C.
3
```
模式{A, B, C}与右侧元组相匹配。

另一种形式：

```
1> {point, X, Y} = {point, 1, 2}.
{point,1,2}
2> X.
1
3> Y.
2
4> 

```

此处，模式要求元组的第一个元素必须是原子point（用作标签）。

如果对应字段不相等，匹配就会失败：

```
5> {x, x} = {2, 1}. 
** exception error: no match of right hand side value {2,1}
```

