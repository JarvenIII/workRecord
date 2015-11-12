原文地址： http://es6.ruanyifeng.com/#docs
作者： 阮一峰

## ECMAScript 6简介
ECMAScript 6（以下简称ES6）是JavaScript语言的下一代标准，它的目标，是使得JavaScript语言可以用来编写复杂的大
型应用程序，成为企业级开发语言。
## let和const命令
### 1. let命令
let类似于var，用来声明变量，但是只在命令所在的代码块有效。解决了JavaScript没有块级作用域的缺陷。
for循环就很适合使用let命令。

```
var a = [];
for (i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); 
```
如果i使用var定义，那么a\[6\]() = 10, 因为i经过var声明后在全局有效，经过循环后值保留为10；如果用let定义，那么
a\[6\]() = 6。应该是每次循环都是一个块，var会将前面的覆盖，而let不会。

#### 变量提升
“变量提升”：变量可以在声明前使用。var会造成这种现象，而let不会，所以typeof不再是一个100%
安全的操作。

现在是不是在块级作用域里默认用let声明？

#### 暂时性死区
只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。
```
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}

```
### 2. 块级作用域
ES5没有块级作用域，缺点1：内层变量会覆盖外层变量
函数提升：不管是否执行if代码块，函数和变量声明都会提升到当前作用域顶部。
```
var tmp = new Date();
function f(){
  console.log(tmp);
  if (false){
    var tmp = "hello world";
  }
}
f() // undefined 
```
缺点2：用于计数的循环变量泄露为全局变量。
#### ES6的块级作用域
外层作用域不能读取内层作用域的变量。
块级作用域可以任意嵌套。
内层作用域可以定义外层作用域的同名变量。
需要注意的是，如果在严格模式下，函数只能在顶层作用域和函数内声明，其他情况（比如if代码块、循环代码块）的声明都会报错。

### 3. const命令
声明常量，不能修改；一旦声明，必须立即赋值。
const的作用域和let相同，只在声明所在的块级作用域内有效。
const声明的变量也不提升，同样存在暂时性死区。
不可重复声明：
```
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```
对于复合类型的变量比如数组，const只是指向数据所在的地址。
```
const foo = {};
foo.prop = 123;

foo.prop // 123
foo = {} // TypeError: "foo" is read-only不起作用

const a = [];
a.push("Hello"); // 可执行
a.length = 0;    // 可执行
a = ["Dave"];    // 报错
```
ES5只有两种声明变量的方法：var和function；Es6有6种：let，const，import，class。

### 4. 跨模块常量
```
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```
### 5. 全局对象的属性
ES5之中，全局对象的属性与全局变量是等价的。
ES6为了改变这一点，一方面规定，var命令和function命令声明的全局变量，依旧是全局对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于全局对象的属性。

## 变量的解构赋值
### 1. 数组的解构赋值
ES6允许从数组和对象中提取值，对变量进行赋值，这被称为解构。
```
var [a, b, c] = [1, 2, 3];
```
本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。
如果解构不成功，变量的值就等于undefined。
```
let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```
另一种情况是不完全解构
```
let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```
如果等号的右边不是数组（或者严格地说，不是可遍历的结构，参见《Iterator》一章），那么将会报错。
```
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```
上面的表达式都会报错，因为等号右边的值，要么转为对象以后不具备Iterator接口（前五个表达式），要么本身就不具备Iterator接口（最后一个表达式）。也就是说，只要某种数据结构具有Iterator接口，都可以采用数组形式的解构赋值。
#### 默认值
ES6内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
```
var [x = 1] = [undefined];
x // 1

var [x = 1] = [null];
x // null

let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```
### 2. 对象的解构赋值
数组的元素是按次序排列的，而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。如果变量名与属性名不一致，必须写成下面这样。
```
let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```
这实际上说明，对象的解构赋值是下面形式的简写：
```
var { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
```
注意，采用这种写法时，变量的声明和赋值是一体的。对于let和const来说，变量不能重新声明，所以一旦赋值的变量以前声明过，就会报错。
```
let foo;
let {foo} = {foo: 1}; // SyntaxError: Duplicate declaration "foo"
```
下面是嵌套赋值的例子。
```
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] }) = { foo: 123, bar: true }; //为什么加个()

obj // {prop:123}
arr // [true]
```
因为JavaScript引擎会将{x}理解成一个代码块，从而发生语法错误。只有不将大括号写在行首，避免JavaScript将其解释为代码块，才能解决这个问题。
如果解构失败，变量的值等于undefined。
对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。
```
let { log, sin, cos } = Math;
```
### 3. 字符串的解构赋值
字符串被转换成了一个类似数组的对象。
```
const [a, b, c, d, e] = 'hello';
let {length : len} = 'hello';
len // 5
```
### 4. 数值和布尔值的解构赋值
解构赋值时，如果等号左边是对象，右边是数值和布尔值，则会先转为对象(前面已经提到左边是数组那么会报错)。
解构赋值的规则是，只要等号右边的值不是对象，就先将其转为对象。由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。
```
let {toString: s} = 123;
s === Number.prototype.toString // true
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```
### 5. 函数参数的解构赋值
函数的参数也可以使用解构赋值。
```
[[1, 2], [3, 4]].map(([a, b]) => a + b) // [ 3, 7 ]
```

注意写法不同会得到不同的结果：
```
function move({x = 0, y = 0} = {}) {
  return [x, y];
} //参数是一个对象，对变量x,y解构赋值

function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
} //对参数指定默认值

move({x: 3, y: 8}); // [3, 8] [3, 8]
move({x: 3}); // [3, 0] [3, undefined]
move({}); // [0, 0] [undefined, undefined]
move(); // [0, 0] [0, 0]
```
### 6. 圆括号问题
建议只要有可能，就不要在模式中放置圆括号。
（1）变量声明语句、函数参数中，模式不能带有圆括号。
（2）不能将整个模式，或嵌套模式中的一层，放在圆括号之中。
```
// 全部报错
var { o: ({ p: p }) } = { o: { p: 2 } };
function f([(z)]) { return z; }
({ p: a }) = { p: 42 };
[({ p: a }), { x: c }] = [{}, {}];
```
### 7. 用途
（1）交换变量的值 [x, y] = [y, x];
（2）从函数返回多个值
```
// 返回一个数组
function example() {
  return [1, 2, 3];
}
var [a, b, c] = example();

// 返回一个对象
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
var { foo, bar } = example();
```
（3）提取JSON数据
```
var jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
}
let { id, status, data: number } = jsonData;
console.log(id, status, number)
// 42, OK, [867, 5309]
```
（4）遍历Map结构
```
var map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
for (let [key] of map) {
for (let [,value] of map) {
```
## class
### 1. 基本语法
ES6的类，完全可以看作构造函数的另一种写法。
