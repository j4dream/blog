# 常见实现
## new 运算符实现
使用new命令的时候，后面的函数就不是正常调用，会执行以下步骤。  
1.创建一个空对象，作为要返回的对象。  
2.将空对象的原型，指向构造函数的prototype属性。  
3.将空对象赋值给函数内部的this。//constructor.apply(obj, args);   
4.执行构造函数内部的代码。
```javascript
function _new(/* 构造函数 */ constructor, /* 构造函数参数 */ param1) {
  // 将 arguments 对象转为数组
  var args = [].slice.call(arguments);
  // 取出构造函数
  var constructor = args.shift();
  // 创建一个空对象，继承构造函数的 prototype 属性
  var context = Object.create(constructor.prototype);
  // 执行构造函数
  var result = constructor.apply(context, args);
  // 如果返回结果是对象，就直接返回，则返回 context 对象
  return (typeof result === 'object' && result != null) ? result : context;
}

// 实例
var actor = _new(Person, '张三', 28);
```

## call, apply, bind 实现
```javascript
// call
Function.prototype.customCall = function() {
  const args = [...arguments];

  const ctx = args.shift() || window;

  ctx.fun = this;

  const res = args.length > 0 ? ctx.fun(...args) : ctx.fun();

  delete ctx.fun;

  return res;
}

// apply

Function.prototype.customApply = function() {
  const args = [...arguments];

  const ctx = args.shift() || window;

  ctx.fun = this;

  const res = args.length > 0 ? ctx.fun(...args[1]) : ctx.fun();

  delete ctx.fun;

  return res;
}

// bind

Function.prototype.customBind = function() {
  const args = [...arguments];

  const ctx = args.shift() || window;

  ctx.fun = this;

  return function() {
    // bind 调用时，和实际调用时参数合并
    const allArgs = args.concat([...arguments]);

    return ctx.fun(...allArgs);
  }
}

```

## throttle debounce
```javascript
// debounce tail
function debounce(fun, delay = 100, ctx) {
  let tid = null;

  return () => {
    clearTimeout(tid);
    tid = setTimeout(() => fun.apply(ctx, [...arguments]), delay);
  }
}

// debounce start
function debounce(fun, delay = 100, ctx) {
  let tid = null;
  let immed = true;
  return () => {
    if (immed) {
      fun.apply(ctx, [...arguments]);
      immed = false;
    }

    clearTimeout(tid);

    tid = setTimeout(() => immed = true, delay);

  }
}

// throttle

function throttle(fun, delay = 100, ctx) {
  let isLive = true;
  return () => {
    if (isLive) {
      fun.apply(ctx, [...arguments]);
      isLive = false;

      setTimout(() => isLive = true, delay);
    }
  }
}
```
## 并集，差集，交集
```javascript
let set1 = new Set([1, 2, 3])
let set2 = new Set([4, 3, 2])

let intersect = new Set([...set1].filter(value => set2.has(value)))
let union = new Set([...set1, ...set2])
let difference = new Set([...set1].filter(value => !set2.has(value)))

console.log(intersect)	// Set {2, 3}
console.log(union)		// Set {1, 2, 3, 4}
console.log(difference)	// Set {1}
```