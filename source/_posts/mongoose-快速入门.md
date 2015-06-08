title: mongoose 快速入门
date: 2014-11-11 18:51:18
tags: [mongodb, mongoose, 学习笔记]
---

### Getting Started

 安装什么的，[戳这里](http://f10.moe/2014/11/11/mongodb-robomongo-mongoose/) 。

 新建一个 test.js， 并且 require mongoose 模块以及连接 test 数据库。

```js

var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

```

---

 当然，需要检测连接数据库时数据库是否有报错，于是就可以用回调的方式来操作：

```
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function callback () {
	  // 成功连接，并进行相应操作
});
```

---

 假设，现在连接成功了(所以接下来的代码是写在 `db.once('open', callback(){...})` 的回调里面的)。 在 Mongoose 中， 任何一切都是来自于 [Schema](http://mongoosejs.com/docs/guide.html), 所以就获取它的引用并且定义我们自己的结构吧！（也可以把这里认为是在定义表的结构）
```
var Testschema = mongoose.Schema({
  name: String
});
```

---

 现在已经新建了一个 Testschema， 接下来把它用到 [Model](http://mongoosejs.com/docs/models.html) 中。（可以认为是在定义 Testschema 结构的表）

```
var Test = mongoose.model('Test', Testschema);

```

---

 这个 Test 是构建文件的类，接下来创建一个 Test 的实例， 就可以了0. 0（可以认为是在定义要插入等操作的数据）

```
var test  = new Test({ name: 'miao' })
console.log(test.name) // 'miao'
```

---

当然还可以给 Testschema 添加方法：

```
Testschema.methods.say = function() {
  var name = this.name
    ? "My name is " + this.name
    : "I don't know my name.";
  console.log(name);
}

var Test= mongoose.model('Test', Testschema);

```

然后再试试实例化：

```
var test = new Test({name : 'minary'});
test.say(); //My name is minary

var test0 = new Test({});
test0.say(); //I don't know my name.
```

---

咦，怎么还没涉及到任何 MongDB 的操作！别急！任何实例都可以直接用 `save` 方法来存到数据库里。第一个回调回来的参数为 `ERROR`。

```
test.save(function(err, data){
  if(err) return console.error(err);	
});
```

可以去数据库看一下哦0. 0，已经有了一个 `tests` 表和 `name` 为 ‘minary’ 的数据了：

```json
5461f6d380f9d3435bd2ed9d Tue, 11 Nov 2014 11:45:23 GMT
{
    name: "minary",
    _id: ObjectId("5461f6d380f9d3435bd2ed9d"),
    __v: 0
}
```

为什么会有 `tests` 这张表呢？你可以再加一条下面的代码试试就知道了：

```
var Test0= mongoose.model('Test0', Testschema);
var test0 = new Test0({name : '@v@'});
test0.save(function(err, data){
  if(err) return console.error(err);    
});
```

---

我们可以通过 Test 来访问所有的记录。

```

Test.find(function(err, data) {
  if(err) return console.error(err);    
  console.log(data);
});

```

---

上面只是把所有的记录都输出来了，那么如果我们要通过 `name` 来查询对应的记录呢！
Mongoose 支持 [MongoDBs](http://mongoosejs.com/docs/queries.html) 里丰富的语法。

像我们现在要查询 `name` 开头为 ‘min’ 的数据：

```
Test.find({name: /^min/}, function(err, data) {
  if(err) return console.error(err);    
  console.log(data);
});

```

---

### Congratulations

快速入门什么的也就结束了， 我们创建一个 schema ， 添加自定义方法，通过 Mongoose 保存和查询 MongoDB 的数据。

这里有更详细的 [guide](http://mongoosejs.com/docs/guide.html) 和 [API docs	](http://mongoosejs.com/docs/api.html) .

### link

http://mongoosejs.com/docs/index.html
