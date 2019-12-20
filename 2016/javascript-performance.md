原文 [JavaScript Performance For The Win](https://dockyard.com/blog/2014/09/22/javascript-performance-for-the-win)  
## 不会被优化的代码 

try-catch 和 try-finally 代码块是不会被v8引擎优化的(如果函数包含try代码块，整个函数都不会被优化)。

```javascript
try {
    willThrowError();
} catch(exception) {
    handleExceptions();
}
```
更好得写法就是把try代码块分离出来，独立写在另一个函数里面。


```javascript
function willThrowError() {
    try {
        // [code]
    } catch(exception) {
        handleExceptions();
    }
}

// code
willThrowError();
// code
```
## 使用本地变量
如果你多次使用一段代码，最好创建局部变量  
1. 快速获得此变量(作用域, 局部作用域的变量,它会更快的检索出来)  
2. 缓存(执行一次操作和存储结果让浏览器做更少的工作)

## 字面量

使用字面量创建对象

```javascript
var arr = [];
var arr = new Array(11);

```
在程序里，你很少知道数组的长度
。让v8引擎自己管理增加的数组元素，它会保证数组在"fast element" 模式下访问数组元素。

### "Dictionary Mode"  
当你动态得添加大量属性(在构造函数外面)到一个对象里面时候，它会进入"Dictionary Mode", 删除属性也会进入"Dictionary Mode"。进入"Dictionary Mode" 的对象将会变慢很多。
[Test Case](https://jsperf.com/test-dictionary-mode)


```javascript
function forInFunc() {
  var dictionary = {'+': 5};
  for (var key in dictionary);
}
```
当您将对象当作字典使用时，它将变成字典（哈希表）。 将这样的对象传递给for-in是不可以的。

### 遍历数组

```javascript
function arrayFunc() {
  var arr = [1, 2, 3];
  for (var index in arr) {

  }
}
```
遍历数组使用 for-in 要比 for 循环慢得多，如果函数包含 for-in 声明则不会被优化。

需要遍历对象属性？

```javascript
var objKeys = Object.keys(obj);
var propertyName;

for (var i = 0, l = objKeys.length; i < l; i++) {
  propertyName = objKeys[i];
  // more code
}

for (var propertyName in obj) {
  if (obj.hasOwnProperty(propertyName)) {
    // more code
  }
}
```
### For-In

For-In statements can prevent the entire function from being optimized in a few cases. It will result in "Not optimized: ForInStatement is not fast case" bailout.

### key has to be a pure local variable

It cannot be from upper scope or referenced from lower scope.

```javascript
var key;

function doesNotSeemToBeLocalKey() {
  var obj = {};
  for (key in obj);
}
```

### Arguments
粗略地使用参数的操作可能导致整个函数是不可优化的。它可能导致这些"bailouts"之一， "未优化： 错误的 arguments 值"：和 "未优化，赋值给arguments对象中得参数"。

### 重新分配参数

```javascript
// 不要给参数重新赋值
function argumentsReassign(foo, bar) {
  if (foo && foo === 5) {
    bar = 'Barracudas';
  }
  // code that uses `bar`
}

// 使用本地变量代替
function argumentsReassign(foo, bar) {
  var localBar;

  if (foo && foo === 5) {
    localBar = 'Beantown Pub';
  }
  // code that uses `localBar`
}
```
### 泄漏参数

```javascript
// `arguments'是一个特殊的对象，它的实现是昂贵的。
function leaksArguments() {
  var args = [].slice.call(arguments);
  // code that uses `args`
}

// 不会泄漏参数
// 访问`arguments.length`只是一个整数，不会实现`arguments`对象。
function doesNotLeakArguments() {
  var args = new Array(arguments.length);

  for (var i = 0; i < args.length; ++i) {
    // `i` is always valid index in the arguments object
    // so we merely retrieve the value
    args[i] = arguments[i];
  }
  // code that uses `args`
}
```
注意，大多数情况下 优化意味更多得代码，可以将上面代码封装一下：

```javascript
function doesNotLeakArguments() {
  arguments_slice(args, arguments);
  // code that uses `args`
}
```
Happy code!
