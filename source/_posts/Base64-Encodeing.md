title: Codewars# Base64 Encodeing
date: 2014-10-14 13:03:55
tags: [javascript,node,codewars]
---

## Description:

[题目链接](http://www.codewars.com/kata/5270f22f862516c686000161)

Extend the String object to create a function that converts the value of the String to and from Base64 using the ASCII character set.



-------------

## Usage:

```
// should return 'dGhpcyBpcyBhIHN0cmluZyEh'
'this is a string!!'.toBase64(); 

// should return 'this is a string!!'
'dGhpcyBpcyBhIHN0cmluZyEh'.fromBase64();
```


You can learn more about Base64 encoding and decoding [here](http://en.wikipedia.org/wiki/Base64).

-------------

## Solutions:

从wiki里可以看出，字符串到base64的转换步骤有四步：

1. 将字符转化成ASCII （取三个字符）
2. 将这三个ASCII转换成二进制（8位）
3. 将这三个二进制（3 × 8 位）转换成四个二进制（4 × 6位）
4. 根据这四个二进制去 *Base64 encoding* 表读取对应的位置的字符


嘛嘛！语言组织的不是很理想，可以直接[戳这里](http://en.wikipedia.org/wiki/Base64).

---


**用现有的方法可以直接用下面的代码实现：**

```

String.prototype.toBase64 = function() {
  return new Buffer(this.toString()).toString('base64');
}

String.prototype.fromBase64 = function() {
  return new Buffer(this.toString(), 'base64').toString();
}

```

**具体实现**

找了个讨论里面最推荐的方法， 具体实现都是大同小异。

```
String.prototype.toBase64 = function() {
  var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
  var encoded = '';
  
  for(var i=0; i < this.length; i+=3) {
    var c1 = this.charCodeAt(i), 
        c2 = this.charCodeAt(i+1), 
        c3 = this.charCodeAt(i+2);
    encoded += chars[(c1 & 0xFC) >> 2];
    encoded += chars[((c1 & 0x03) << 4) | ((c2 & 0xF0) >> 4)];
    encoded += chars[((c2 & 0x0F) << 2) | ((c3 & 0xC0) >> 6)];
    encoded += chars[c3 & 0x3F];
  }
  
  return encoded;
};

String.prototype.fromBase64 = function() {
  var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
  var decoded = '';
  
  for(var i=0; i < this.length; i+=4) {
    var c1 = chars.indexOf(this[i]), 
        c2 = chars.indexOf(this[i+1]), 
        c3 = chars.indexOf(this[i+2]),
        c4 = chars.indexOf(this[i+3]);
    decoded += String.fromCharCode(((c1) << 2) | ((c2 & 0x30) >> 4));
    decoded += String.fromCharCode(((c2 & 0x0F) << 4) | ((c3 & 0xFC) >> 2));
    decoded += String.fromCharCode(((c3 & 0x03) << 6) | c4);
  }
  
  return decoded;
};

```

**解释上面的代码：**

1. Base64 cording 表：

        var chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';

2. 这段代码的意思是取出三个字符，并将他们转换成ASCII码：

        var c1 = this.charCodeAt(i), 
        c2 = this.charCodeAt(i+1), 
        c3 = this.charCodeAt(i+2);

3. 字符处理：

        encoded += chars[(c1 & 0xFC) >> 2];
        encoded += chars[((c1 & 0x03) << 4) | ((c2 & 0xF0) >> 4)];
        encoded += chars[((c2 & 0x0F) << 2) | ((c3 & 0xC0) >> 6)];
        encoded += chars[c3 & 0x3F];

``(c1 & 0xFC) >> 2`` ：*0xFC* 为十六进制，转换为二进制为 *11111100* , 将 *c1*  **按位与** *11111100* （&是按位与）, 结果就是高6位保持不变，低两位清0。**>>2** 是右移两位，就是把高6位移到低6位。

``((c1 & 0x03) << 4) | ((c2 & 0xF0) >> 4)``： *0x03* 转换为二进制为 *00000011* , 所以 *((c1 & 0x03) << 4)* 就是前6位清零后两位不变， 再左移4位。 

   假设 *c1* 为 *01001101*, 经过第一步后变为 *00000001*， 再经过左移后变为 *00010000* 。

   假设 *c2* 为 *01100001* , 经过 & 并右移4位为： *00000110* 。 *c1* | *c2* : *00010110* ，也就是 *010110* 。

   其他的都是这么一个意思就不解释了。还有，解码就是上面的逆运算. 嘛嘛！你们肯定会的0. 0 。


## About 

  由于上面涉及了 *位运算* , 那就顺便提一下。
  
  *朴灵* 的《深入浅出》有提到：
  
  Javascript 的一个典型弱点就是 *位运算* 。Javascript 的位运算是参照 *Java* 的位算法实现的， 但是 *Java* 位运算是在 int 型数字的基础上运行的， 而 Javascript 中只有 double 型的数据类型， 在进行位运算的过程中， 需要将 double 型转换成 int 型， 然后再进行。 所以 Javascript 层面上做位运算效率不高。
  
  在应用中， 会频繁出现位运算的需求， 包括 *转码* , *编码* 的过程， 如果通过 Javascript 来实现， CPU 资源将会耗费很多， 这时编写 *C/C++* 扩展模块来提升性能的机会就来了。
  