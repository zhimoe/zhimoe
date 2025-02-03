+++
title = 'Modern Javascript Features'
date = '2025-01-12T15:03:05+08:00'
categories = ['编程']
tags = ['code','js']
toc = true
+++

一些很甜的 JS 语法糖。

<!--more-->

### Nullish Coalescing Operator (??) 
当值为 nullish 时使用默认值。注意，nullish 和 truthy 不一样。
nullish: null or undefined.
```js
const foo = null ?? 'default string';
console.log(foo);
// Expected output: "default string"

const baz = 0 ?? 42;
console.log(baz);
// Expected output: 0
```

### Nullish coalescing assignment (??=)
当变量为 nullish 时则赋值，否则无变化。
```js
const a = { duration: 50 };

a.speed ??= 25;
console.log(a.speed);
// Expected output: 25

a.duration ??= 10;
console.log(a.duration);
// Expected output: 50
```
### Logical OR/AND assignment (||= &&=)
使用场景比较少。 
```js
const a = { duration: 50, title: '' };

a.duration ||= 10;
console.log(a.duration);
// Expected output: 50

a.title ||= 'title is empty.';
console.log(a.title);
// Expected output: "title is empty."

// &&= 使用场景比较少
let a = 1;
let b = 0;

a &&= 2;
console.log(a);
// Expected output: 2

b &&= 2;
console.log(b);
// Expected output: 0
```
### Optional chaining (?.)
kotlin 笑而不语。超级有用的语法糖。
既可以访问对象的属性也可以访问 function。
注意如果变量是 undefined/null 时返回的是 undefined 而不是 null。
```js
const adventurer = {
  name: 'Alice',
  cat: {
    name: 'Dinah',
  },
};

const dogName = adventurer.dog?.name;
console.log(dogName);
// Expected output: undefined

// call a function
console.log(adventurer.someNonExistentMethod?.());
// Expected output: undefined
```

### Dynamic Import
import() 不是 function call，只是语法相似。所以不能使用 apply/call 等语法。
import 不需要指定<script type="module".>
```js
<script>
  async function load() {
    let say = await import('./say.js');
    say.hi(); // Hello!
    say.bye(); // Bye!
    say.default(); // Module loaded (export default)!
  }
</script>
```

### Private properties
变量名不能是 `#constructor`;
在 chrome devtool 中可以在外部访问 private properties，只是一个 chrome 开发特性，不是语法问题。
[MDN doc:private properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_properties)
```js
class ClassWithPrivate {
  #privateField;
  #privateFieldWithInitializer = 42;

  #privateMethod() {
    // …
  }

  static #privateStaticField;
  static #privateStaticFieldWithInitializer = 42;

  static #privateStaticMethod() {
    // …
  }
}

const instance = new ClassWithPrivateField();
instance.#privateField; // Syntax error

```

### error.cause
error 新增的属性，用来在 error 中传递原始错误信息。
```js
try {
    conn = getDbConnection();
  } catch (err) {
    throw new Error('Connecting to database failed.', { cause: err });
}
```
### Array functions

```js
// arr.toSorted 
const values = [1, 10, 21, 2];
const sortedValues = values.toSorted((a, b) => a - b);

// findLast
const found = array1.findLast((element) => element > 45);
// toReversed()
// toSpliced(start, deleteCount, item1, item2, /* …, */ itemN)

// with return new array
const arr = [1, 2, 3, 4, 5];
console.log(arr.with(2, 6)); // [1, 2, 6, 4, 5]

// Array.fromAsync() static method 
Array.fromAsync(
  new Set([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)]),
).then((array) => console.log(array));
// [1, 2, 3]
```
  Array.fromAsync() and Promise.all() can both turn an iterable of promises into a promise of an array. However, there are two key differences:

  Array.fromAsync() awaits each value yielded from the object sequentially. Promise.all() awaits all values concurrently.

  Array.fromAsync() iterates the iterable lazily, and doesn't retrieve the next value until the current one is settled. Promise.all() retrieves all values in advance and awaits them all.

### apply,call, bind
三个方法都是用于修改函数本身的的 context。
```js
call(this,arg1,arg2,arg3)
// 将一个 array like（有元素，length 方法但是没其他方法）对象变成 array
const slice = Array.prototype.slice;
slice.call(arguments);

apply(this,argsArray)
// e.g. const max = Math.max.apply(null, numbers);
// 和 rest parameters 语法非常相似：
function wrapper(...args) {
  return anotherFn(...args);
}
// 使用场景：将一个 array append 到另一个 array,
// array.push 方法只能接收一个元素，如果使用 concat 得到的是一个新的 array
const array = ["a", "b"];
const elements = [0, 1, 2];
array.push.apply(array, elements);
//等价于 array.push(...elements);


bind(this,arg1,arg2,arg3) like call but return the new function.
// 将上面的 slice.call 优化成 slice() 函数
// Same as "slice" in the previous example
// 给 call 方法设置一个 this 对象并返回新的 call 函数，
// 这个 call 函数的 this 是 Array.prototype.slice;
const slice = Function.prototype.call.bind(Array.prototype.slice);
slice(arguments);
// 为什么不是 Array.prototype.slice.call.bind(Array.prototype.slice) ?
```