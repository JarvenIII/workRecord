## ECMAScript 6简介
ECMAScript 6（以下简称ES6）是JavaScript语言的下一代标准，它的目标，是使得JavaScript语言可以用来编写复杂的大
型应用程序，成为企业级开发语言
## let和const命令
### let命令
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
// 如果i使用var定义，那么a\[6\]() = 10, 因为i经过var声明后在全局有效，经过循环后值保留为10；如果用let定义，那么
a\[6\]() = 6，调用过程直接循环？

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
### 块级作用域
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
缺点2：用于计的循环变量泄露为全局变量
#### ES6的块级作用域
外层作用域不能读取内层作用域的变量。
块级作用域可以任意嵌套。
内层作用域可以定义内层作用域的同名变量。
需要注意的是，如果在严格模式下，函数只能在顶层作用域和函数内声明，其他情况（比如if代码块、循环代码块）的声明都会报错。

### const命令
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

## 跨模块常量
