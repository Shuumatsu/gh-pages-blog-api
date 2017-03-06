---
title: Iterator and Generator
categories: [JavaScript, async]
tags: [JavaScript, async]
date: 2016/10/13 20:46:25
---

iterator 与 generator 以及后续的 async await

<!-- more -->


## iterator

### 可遍历（可迭代）协议
一个对象为了变成可遍历对象，比如说可以用 `for ... in` 结构遍历其属性值，必须实现 `@@iterator` 方法, 意思是这个对象（或者它原型链 `prototype chain` 上的某个对象）必须有一个名字是 `Symbol.iterator` 的属性。
|属性|值|
|---|---|
|`[Symbol.iterator]`|返回一个对象的无参函数，被返回对象符合可遍历协议。|

### 迭代器协议
当一个对象被认为是一个迭代器时，它实现了一个 `next()` 的方法。
该方法返回一个对象包含 `done` 和 `value` 属性，`done` 的值表示迭代器是否可以产生序列中的下一个值，`value` 为迭代器返回的任何 `JavaScript` 值。`done` 为 `true` 时可省略。

---

```
let a = {
    q: 'q',
    w: 'w',
    e: 'e',
};
Object.defineProperty(a, length, {
    enumerable: false,
    value: 3
});
```
现在对 a 尝试用 `for ... in` 结构遍历其属性值
```
for (let v of a) {
    console.log(v);
}
```
报错：
```
Uncaught TypeError: a[Symbol.iterator] is not a function
```

定义一个函数利用闭包实现一个将 `a` 转变为可迭代对象
```
function Iterator(obj) {
    let i;

    return ()=> {
        return {
            next: ()=> {
                if (i < obj.length) {
                    for (i in obj) {
                        return {
                            done: false,
                            value: obj[i]
                        };
                    }
                }
                return {
                    done: true
                };
            }
        };
    };
}

a[Symbol.iterator] = Iterator(a);
```

## generator
`Generator` 函数最大特点就是可以交出函数的执行权（即暂停执行）。异步操作需要暂停的地方，都用 yield 语句注明。调用 `Generator` 函数并不会执行本体，而是每次调用 `next` 方法的时候，执行到下一个碰到的 `yield` 处。


### Generator 函数的数据交换

```
function* anotherGenerator(i) {
  yield i + 1;
  let x = yield i + 2;
  yield x + 3;
}

function* generator(i){
  yield i;
  yield* anotherGenerator(i);
  // 执行权转交给另一个 generator 函数的话 yield 后面带星号，直接调用的话是没有效果的
  yield i + 10;
}

var gen = generator(10);

console.log(gen.next().value); // 10
console.log(gen.next().value); // 11
console.log(gen.next().value); // 12
console.log(gen.next(2).value); // 5
console.log(gen.next().value); // 20
// next 的返回值和迭代器的 next 类似，yield 语句的执行结果作为 value，是否还有下一个 yield 决定 done
// next 方法可以带有参数，这个参数可以传入 Generator 函数，作为上个阶段异步任务的返回结果
```

### Generator 函数的错误处理
`Generator` 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。
```
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){ 
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
g.throw（'出错了'）;
// 出错了
```
上面代码的最后一行，`Generator` 函数体外，使用指针对象的 `throw` 方法抛出的错误，可以被函数体内的 `try ... catch` 。

### Generator 函数的终止
Generator函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数。
```
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```
如果Generator函数内部有try...finally代码块，那么return方法会推迟到finally代码块执行完再执行。
```
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers()
g.next() // { done: false, value: 1 }
g.next() // { done: false, value: 2 }
g.return(7) // { done: false, value: 4 }
g.next() // { done: false, value: 5 }
g.next() // { done: true, value: 7 }
```
上面代码中，调用return方法后，就开始执行finally代码块，然后等到finally代码块执行完，再执行return方法。

### Generator 函数的自动执行
co 库是tj写的一个让Generator函数自动执行的工具。
```
let co = require('co');
let p = co(gen);

p.then(function (){
  console.log('ok');
})
```
co 函数返回一个 Promise 对象，因此可以用 then 方法添加回调函数。

#### async 与 await
ES7 为 generator 的语法糖