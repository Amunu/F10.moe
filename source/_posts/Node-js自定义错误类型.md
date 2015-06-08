title: Node.js自定义错误类型
date: 2014-11-05 14:33:55
tags: [node, error]
---

最近的项目里用到了自定义错误类型，因此也就深入了解一下。

### Subclassing Error

首先我们可以定义一个 Error 的子类。通过 `Object.create` 和 `util.inherits` 很容易实现：

```
var assert = require('assert');
var util = require('util');

function NotFound(msg){
  Error.call(this);
  this.message = msg;
}
util.inherits(NotFound, Error);
var error = new NotFound('not found');
assert(error.message);
assert(error instanceof NotFound);
assert(error instanceof Error);
assert.equal(error instanceof RangeError, false);
```

可以通过 `instanceof` 来检查错误类型，根据类型进行不同的处理。
上面的代码设置了自带的message， 并且 error 是 NotFound 和 Error 的一个实例， 但是不是 RangeError。

如果用了 [express](http://expressjs.com/) 框架， 就能设置其他的 properties 让 error 变得更有用。比方说当处理一个HTTP的错误时， 就可以写成这样：

```
function NotFound(msg) {
  Error.call(this);
  this.message = msg;
  this.statusCode = 404;
}
```

现在就已经可以通过错误处理的中间件来处理错误信息：

```
app.use(function(err, req, res, next) {
  console.error(err.stack);

  if (!err.statusCode || err.statusCode === 500) {
    emails.error({ err: err, req: req });
  }

  res.send(err.statusCode || 500, err.message);
});
```

这会发送HTTP的状态码给浏览器， 当 err 的 statusCode 未设置或者等于 500 的时候， 就通过邮件来发送这个错误。这样就能排除那些 404， 401， 403等等的错误。

读取 ``console.error(err.stack)`` 事实上并不会像预期那样工作，像 node， chrome 基于 V8 的可以使用 `Error.captureStackTrace(this, arguments.callee)` 的错误构造函数来进行堆栈跟踪。

```
var NotFound = function(msg) {
  Error.call(this);
  Error.captureStackTrace(this, arguments.callee);
  this.message = msg || 'Not Found';
  this.statusCode = 404;
  this.name = "notFound"
}
util.inherits(NotFound, Error);

export.NotFoundError = NotFound;
```

当然我们还可以将上面这个创建的抽象错误类型扩展到其他自定义错误中：

```
var notFountError = require('./error').NotFountError; 
var UserNotFound = function(msg){
  this.constructor.super_(msg);
}

util.inherits(UserNotFound, notFoundError);

```

### link

http://dailyjs.com/2014/01/30/exception-error/
https://cnodejs.org/topic/52090bc944e76d216af25f6f
