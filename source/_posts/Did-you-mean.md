title: Codewars#Did you mean ...?
date: 2014-10-15 18:11:47
tags: [codewars, javascript, string]
---

## Description:
[题目链接](http://www.codewars.com/kata/5259510fc76e59579e0009d4)
I'm sure, you know Google's "Did you mean ...?", when you entered a search term and mistyped a word. In this kata we want to implement something similar.

You'll get an entered term (lowercase string) and an array of known words (also lowercase strings). Your task is to find out, which word from the dictionary is most similar to the entered one. The similarity is described by the minimum number of letters you have to add, remove or replace in order to get from the entered word to one of the dictionary. The lower the number of required changes, the higher the similarity between each two words.

Same words are obviously the most similar ones. A word that needs one letter to be changed is more similar to another word that needs 2 (or more) letters to be changed. E.g. the mistyped term berr is more similar to beer (1 letter to be replaced) than to barrel (3 letters to be changed in total).

Extend the dictionary in a way, that it is able to return you the most similar word from the list of known words.

## Code Examples:

```
fruits = new Dictionary(['cherry', 'pineapple', 'melon', 'strawberry', 'raspberry']);
fruits.findMostSimilar('strawbery'); // must return "strawberry"
fruits.findMostSimilar('berry'); // must return "cherry"
---
things = new Dictionary(['stars', 'mars', 'wars', 'codec', 'codewars']);
things.findMostSimilar('coddwars'); // must return "codewars"
---
languages = new Dictionary(['javascript', 'java', 'ruby', 'php', 'python', 'coffeescript']);
languages.findMostSimilar('heaven'); // must return "java"
languages.findMostSimilar('javascript'); // must return "javascript" (same words are obviously the most similar ones)
I know, many of you would disagree that java is more similar to heaven than all the other ones, but in this kata it is ;)
```

## Additional notes:

there is always exactly one possible solution

## Solutions:

解决这题的方法是用一个已经存在的算法， 叫做Levenshtein距离，又称[编辑距离](http://en.wikipedia.org/wiki/Levenshtein_distance) 。

具体的步骤就是：（假设两个字符串为 'heaven' ，'java'）

 **1.建一个二位数组， 如下表所示：**

<table class="table table-striped-white table-bordered"><thead><tr><th>0</th><th style="text-align:center;">j(1)</th><th style="text-align:center;">a(2)</th><th style="text-align:center;">v(3)</th><th>a(4)</th></tr></thead><tbody><tr><td>h(1)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(2)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>a(3)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>v(4)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(5)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>n(6)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr></tbody></table>

 **2.第二步见下图：**

<table class="table table-striped-white table-bordered"><thead><tr><th>0</th><th style="text-align:center;">j(1)</th><th style="text-align:center;">a(2)</th><th style="text-align:center;">v(3)</th><th>a(4)</th></tr></thead><tbody><tr><td>h(1)</td><td style="text-align:center;">A</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(2)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>a(3)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>v(4)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(5)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>n(6)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr></tbody></table>

这时需要算出A的值，若A的左边和A的上边对应的字母不一致， 像上表的 'h' 和 'j' ，则取< *斜上角的数值 + 1* ， 上面的 *数值 + 1* , 左边的 *数值 + 1* > ， 取三个数之间的最小数。   

得出下表的解：(0 + 1 < 1 + 1 <= 1 + 1)

<table class="table table-striped-white table-bordered"><thead><tr><th>0</th><th style="text-align:center;">j(1)</th><th style="text-align:center;">a(2)</th><th style="text-align:center;">v(3)</th><th>a(4)</th></tr></thead><tbody><tr><td>h(1)</td><td style="text-align:center;">1</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(2)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>a(3)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>v(4)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(5)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>n(6)</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr></tbody></table>

 **3.一直按照第 2 步执行下来，可得下面这张表：**

<table class="table table-striped-white table-bordered"><thead><tr><th>0</th><th style="text-align:center;">j(1)</th><th style="text-align:center;">a(2)</th><th style="text-align:center;">v(3)</th><th>a(4)</th></tr></thead><tbody><tr><td>h(1)</td><td style="text-align:center;">1</td><td style="text-align:center;">2</td><td style="text-align:center;"></td><td></td></tr><tr><td>e(2)</td><td style="text-align:center;">2</td><td style="text-align:center;">2</td><td style="text-align:center;"></td><td></td></tr><tr><td>a(3)</td><td style="text-align:center;">3</td><td style="text-align:center;">B</td><td style="text-align:center;"></td><td></td></tr><tr><td>v(4)</td><td style="text-align:center;">4</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>e(5)</td><td style="text-align:center;">5</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr><tr><td>n(6)</td><td style="text-align:center;">6</td><td style="text-align:center;"></td><td style="text-align:center;"></td><td></td></tr></tbody></table>

当执行到B时，java 的 'a' 与 'heaven' 的 'a' 相等。

这时B的值，则取< *斜上角的数值 + 0* ， 上面的 *数值 + 1* , 左边的 *数值 + 1* > ， 取三个数之间的最小数。  也就是 min (2 + 0, 2 + 1， 3 + 1) ， 也就是 B = 2 。

**最终效果：**

<table class="table table-striped-white table-bordered"><thead><tr><th>0</th><th style="text-align:center;">j(1)</th><th style="text-align:center;">a(2)</th><th style="text-align:center;">v(3)</th><th>a(4)</th></tr></thead><tbody><tr><td>h(1)</td><td style="text-align:center;">1</td><td style="text-align:center;">2</td><td style="text-align:center;">3</td><td style="text-align:center;">3</td></tr><tr><td>e(2)</td><td style="text-align:center;">2</td><td style="text-align:center;">2</td><td style="text-align:center;">3</td><td style="text-align:center;">4</td></tr><tr><td>a(3)</td><td style="text-align:center;">3</td><td style="text-align:center;">2</td><td style="text-align:center;">3</td><td style="text-align:center;">3</td></tr><tr><td>v(4)</td><td style="text-align:center;">4</td><td style="text-align:center;">3</td><td style="text-align:center;">2</td><td style="text-align:center;">3</td></tr><tr><td>e(5)</td><td style="text-align:center;">5</td><td style="text-align:center;">4</td><td style="text-align:center;">3</td><td style="text-align:center;">3</td></tr><tr><td>n(6)</td><td style="text-align:center;">6</td><td style="text-align:center;">5</td><td style="text-align:center;">4</td><td style="text-align:center;">4</td></tr></tbody></table>

## Demo:

```
var CalcDistance = function(source, target)
{
    var n = source.length;
    var m = target.length;
    if (m == 0) return n;
    if (n == 0) return m;
    var matrix = [];
    var l = n > m ? n : m;
    for(var i = 0;i <= l;i ++) 		matrix.push([]);
    for (var i = 1; i <= n; i++)	matrix[i][0] = i;
    for (var i = 1; i <= m; i++)	matrix[0][i] = i;
    matrix[0][0] = 0;

    for (var i = 1; i <= n; i++)
    {
        for (var j = 1; j <= m; j++)
        {
            var cost;
            
            if (source[i - 1] == target[j - 1])	cost = 0;
            else cost = 1;

            var above = matrix[i - 1][j] + 1;
            var left = matrix[i][j - 1] + 1;
            var diag = matrix[i - 1][j - 1] + cost;
           	matrix[i][j] = Math.min(above, Math.min(left, diag));
        }
    } 
    return matrix[n][m];
}
```

```
function Dictionary(words) {
  this.words = words;
}

Dictionary.prototype.findMostSimilar = function(term) {
   var mi, nu;
   for (var i = 0; i < this.words.length; i++) {
   	  var count = CalcDistance(this.words[i], term);
   	  if(i === 0) {	mi = i,	nu = count;}
   	  else if(nu > count) {mi = i, nu = count;}
   };
   return(this.words[mi]);
}
```




