title: Mongoose Schemas 深入
date: 2014-11-12 11:13:47
tags: [mongoose, mongodb, 学习笔记]
---

[上一篇](http://f10.moe/2014/11/11/mongoose-%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/)博文有稍稍提了一下 schemas 的定义，这里就结合官网的文档，写写自己的理解。

### Defining your schema

下面这个定义应该是已经没什么问题了，不懂的话可以切到上一篇查看。

```
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var blogSchema = new Schema({
  title:  String,
  author: String,
  body:   String,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,  
  meta: {
    votes: Number,
    favs:  Number
  }
}); 
```

需要注意的是，由于 MongoDB 是非关系型数据库， 所以可以像上面的 `meta` 一样嵌套很多层结构。
被允许的 Schema 类型有：
* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* ObjectId
* Array

### Creating a model

接下来定义一个　`blogSchema` 的模型：

```
var Blog = mongoose.model('Blog', blogSchema);
```

### Instance methods

可以直接用 Mongoose 的查询方法查询比方说 `findOne`等，可以[点这里](http://mongoosejs.com/docs/queries.html)看文档。

现在想自己定义一个查询的方法，如下：

```
// define a schema
var animalSchema = new Schema({ name: String, type: String });
// assign a function to the "methods" object of our animalSchema
animalSchema.methods.findSimilarTypes = function (cb) {
  return this.model('Animal').find({ type: this.type }, cb);
}
var Animal = mongoose.model('Animal', animalSchema);
var dog = new Animal({ type: 'dog' });
dog.findSimilarTypes(function (err, dogs) {
  console.log(dogs); // woof
});
```

### Statics

添加静态方法模型也很简单，继续用 `animalSchema`:

```
animalSchema.statics.findByName = function (name, cb) {
  this.find({ name: new RegExp(name, 'i') }, cb);
}
var Animal = mongoose.model('Animal', animalSchema);
Animal.findByName('fido', function (err, animals) {
  console.log(animals);
});
```
上面的 [RegExp](http://www.w3school.com.cn/jsref/jsref_obj_regexp.asp) 对象表示正则表达式，它是对字符串执行模式匹配的强大工具。

### Indexes

MongoDB 支持[索引](http://docs.mongodb.org/manual/indexes/), 在 Mongoose 里我们在创建 schema 的时候定义索引：

```
var animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true } // field level
});

animalSchema.index({ name: 1, type: -1 }); // schema level
```

---

### link

http://mongoosejs.com/docs/guide.html
