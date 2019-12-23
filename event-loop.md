# 事件循环（Event Loop）

javascript从诞生之日起就是一门单线程脚本语言，单线程意味着，javascript代码在执行的任何时候，都只有一个主线程来处理所有的任务。

## 异步任务
既然是单线程那么如何处理异步任务，这就是事件循环机制了。  
异步任务分为task（宏任务，也可称为macroTask）和microtask（微任务）两类。
当满足执行条件时，task和microtask会被放入各自的队列中等待放入执行线程执行，我们把这两个队列称为Task Queue(也叫Macrotask Queue)和Microtask Queue。

task：script中代码、setTimeout、setInterval、I/O、UI render。
microtask: promise、Object.observe、MutationObserver。


## 执行过程
1. 执行代码，
2. 遇到宏任务，放到宏任务队列中。
3. 遇到微任务，放到微任务队列中。
4. 代码执行完，开始事件循环。
5. 执行完微队列任务，进入下一个循环，如执行过程中又遇到异步任务，参考2， 3。

## 知识巩固
为了更好理解时间循环机制，下面准备了一些题目巩固。

```javascript
function sleep(time) {
  let startTime = new Date()
  while (new Date() - startTime < time) {}
  console.log('1s over')
}
setTimeout(() => {
  console.log('setTimeout - 1')
  setTimeout(() => {
    console.log('setTimeout - 1 - 1')
    sleep(1000)
  })
  new Promise(resolve => resolve()).then(() => {
    console.log('setTimeout - 1 - then')
    new Promise(resolve => resolve()).then(() => {
      console.log('setTimeout - 1 - then - then')
    })
  })
  sleep(1000)
})

setTimeout(() => {
  console.log('setTimeout - 2')
  setTimeout(() => {
    console.log('setTimeout - 2 - 1')
    sleep(1000)
  })
  new Promise(resolve => resolve()).then(() => {
    console.log('setTimeout - 2 - then')
    new Promise(resolve => resolve()).then(() => {
        console.log('setTimeout - 2 - then - then')
    })
  })
  sleep(1000)
})
// 依次输出
setTimeout - 1
1s over
setTimeout - 1 - then
setTimeout - 1 - then - then
setTimeout - 2
1s over
setTimeout - 2 - then
setTimeout - 2 - then - then
setTimeout - 1 - 1
1s over
setTimeout - 2 - 1
1s over
```

```javascript
console.log('1');

setTimeout(function() {
  console.log('2');
  process.nextTick(function() {
    console.log('3');
  })
  new Promise(function(resolve) {
    console.log('4');
    resolve();
  }).then(function() {
    console.log('5')
  })
})
process.nextTick(function() {
  console.log('6');
})
new Promise(function(resolve) {
  console.log('7');
  resolve();
}).then(function() {
  console.log('8')
})

setTimeout(function() {
  console.log('9');
  process.nextTick(function() {
    console.log('10');
  })
  new Promise(function(resolve) {
    console.log('11');
    resolve();
  }).then(function() {
    console.log('12')
  })
})
// 依次输出
1
7
6
8
2
4
3
5
9
11
10
12
```

```javascript
async function async1() {
  console.log(1);
  const result = await async2();
  console.log(3);
  const x = await async2();
  console.log(7);
}

async function async2() {
  console.log(2);
}

Promise.resolve().then(() => {
  console.log(4);
});

setTimeout(() => {
  console.log(5);
});

async1();
console.log(6);
// 输出
1
2
6
4
3
2
7
5
```
## 注意
不同版本 nodejs 结果可能不同，请使用 v11 以上测试，结果跟浏览器一致。
