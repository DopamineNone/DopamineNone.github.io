---
layout: passenge
title: Javascript 笔记
date: 2024-01-20 14:53:19
categories: Javascript
tags: 
    - Javascript
    - Grammar
---

## 概述

本博客仅是本人学习原生Js时的一些记录和总结。  
<span style="color: #ff6d91">注意：</span>`console.log(string)`用于向控制台打印string字符串，是示例中最常用到的代码之一。

## 数据类型及其字面量

EMCAscript规定了7大简单/原始数据类型：

- Boolean 布尔值
- String 字符串
- Number 数值 *包括整型和浮点数*
- null 空值 *注意大小写区分，NULL和Null不是null类型*
- undefined 未定义值
- BigInt 大整型数
- Symbol 一种独特且不可修改的类型

还有Object 这种复杂数据类型

可以使用`typeof x`来判断变量的数据类型

字面量是直接在代码中的固定值，这里记录一些不同数据类型字面量的特点

### 整型与进制（Integer & BigInt）

```Javascript
0, 117, 123456789123456789n             (decimal, base 10, no prefix)
015, 0001, 0o777777777777n              (octal, base 8, prefix： 0 or 0o)
0x1123, 0x00111, 0x123456789ABCDEFn     (hexadecimal, "hex" or base 16, prefix: 0x)
0b11, 0b0011, 0b11101001010101010101n   (binary, base 2, prefix: 0b)
// 以n作为后缀的为BigInt类型
console.log(typeof 0o777777777777n === 'bigint') // true
```

### 浮点数

格式为`[digits].[digits][(E|e)[(+|-)]digits]`,以下为例子：

```Javascript
3.1415926
.123456789
3.1E+12
.1e-23  // 1e-24
```

### 字符串

字符串模板语法：

```Javascript
const name = 'DopamineNone'
const string = `My name is ${name}` // My name is DopamineNone
```

### 数组：多余的逗号

```Javascript
let a = [1, , 3,,]
// a为[1, <empty item>, 3, <empty item>]， 最后一个非空白项后的最后一个逗号会被忽略
// 这里a数组的长度为4，其中访问a[1], a[3]返回的都是undefined
// 但<empty item>不等同于undefined，当对数组a使用遍历函数如map或forEach时，这些空白项会被跳过

// a 的更推荐写法
let a = [1, /* empty */, 3, /* empty */,]
// 或
let a = [1, undefined, 3, undefined]

// 数组字面量支持多行书写
let a = [
    1,
    2,
    3
]
```

### 对象

对象的属性名可以是任何字符串和数字，其中不是合法标识符的属性名只能通过["property-name"]来索引。例子demo如下：

```Javascript
const sales = "Toyota";

function carTypes(name) {
  return name === "Honda" ? name : `Sorry, we don't sell ${name}.`;
}
// 例子
const demo = {
    a: 'Alice',
    'B': carTypes('Alice'), // 函数返回值赋值
    'c': sales, // 变量赋值
    1: 'Bob',
    '':  'Blank',
    '!': 'Exclamation Mark'
}

// right
demo['a'] // Alice
demo.a // Alice
demo.B // Sorry, we don't sell Alice.
demo['B'] // Sorry, we don't sell Alice.
demo[1] // Bob
demo['1'] // Bob
demo[''] // Blank
demo['!'] // Exclamation Mark

// wrong
demo.1
demo.''
demo.'!'
```

对象字面量还允许直接写入__proto__、对象方法，以及支持同名属性简写、属性名计算等

```Javascript
const id = 1
const demo = {
    id, // shorthand for 'id: id'
    __proto__: theProtoObj,
    toString() {
        // Super calls
        return "d " + super.toString();
    },
    // Computed (dynamic) property names
    ["prop_" + (() => 42)()]: 42,
}
```

还有其他数据类型的字面量如Boolean(布尔值)的字面量只有`true`和`false`，<span style="color: #ff6d91">注意大小写！</span>，还有RagExp(正则表达式)、Symbol等类型，会放在后面章节详细讨论。

## 变量

Javascript中想使用变量就得对其先进行声明，其中声明变量有三个方法/关键字：

- var
- let
- const

>三者区别：const用于声明常量，常量变量不可修改值（对象、数组等复杂变量另说），对应的let和var声明的变量可修改值。const和let声明的变量具有块作用域，var没有。使用建议const > let > var。

```Javascript
// 一些例子
if (true) {
  var x = 5; // 无块作用域
  const y = 5; // 有块作用域
}
console.log(x); // x is 5
console.log(y); // ReferenceError: y is not defined
```

>变量提升: 指无论变量在哪被声明，声明都会被提升至其作用域的顶部，但不会同时初始化。const和let声明的变量也存在变量提升，但其将作用域顶部至代码中它被声明的位置被设为“暂时性死区”，在“暂时性死区”使用该变量会抛出ReferenceError。

```Javascript
console.log(typeof v === 'undefined'); // true
var v = 5;

// 以上代码等同于 ->

var v; // 声明但未赋值
console.log(typeof v === 'undefined'); // true
v = 5;

/* ******* 分割线 ******* */

console.log(typeof v === 'undefined'); // ReferenceError
let v = 5;

// 以上代码可理解为于 ->

let v; // 声明但未赋值
console.log(typeof v === 'undefined'); // ReferenceError, 暂时性死区
v = 5;  
```

## 运算

### 常见运算符

- 赋值运算符： =, +=, -=, *=, /=
- 自增运算符： ++, -- *分为前置和后置*
- 比较运算符： >, <, >=, <=, ==(undefined == null is true, 2=='2'is true), ===(*类型和值都相等*, NaN!==NaN is true), !=, !==
- 逻辑运算符： &&, ||, !
- 位运算符：&, |, ^, ~
- 三元运算符： condition ? res1 : res2

>优先级：() >> ++ -- ! >> * / % >> +- >> > >= < <= >> == != === !== >> && >> || >> = >> ,

<span style="color: #ff6d91">注意：</span>NaN表示计算错误，对NaN的任何操作返回NaN

### 其他运算符号

#### delete

delete运算符用于删除对象的属性（极其不建议删除数组元素）,成功返回true，失败返回false

```Javascript
// 示例
delete obj[key]
delete obj.key2

// 返回值
delete Math.PI; // returns false (cannot delete non-configurable properties)

const myObj = { h: 4 };
delete myObj.h; // returns true (can delete user-defined properties)
```

#### typeof

返回变量的数据类型

```Javascript
const myFun = new Function("5 + 2");
const shape = "round";
const size = 1;
const foo = ["Apple", "Mango", "Orange"];
const today = new Date();
const obj = {
    name: 'obj',
    function getName() {
        return this.obj
    }
}

typeof myFun; // returns "function"
typeof shape; // returns "string"
typeof size; // returns "number"
typeof foo; // returns "object"
typeof today; // returns "object"
typeof doesntExist; // returns "undefined"
typeof obj.name // returns "string"
typeof obj.getName // return "function"

// predefined objects
typeof Date; // returns "function"
typeof Function; // returns "function"
typeof Math; // returns "object"
typeof Option; // returns "function"
typeof String; // returns "function"

```

#### in

```Javascript
// Arrays
const trees = ["redwood", "bay", "cedar", "oak", "maple"];
0 in trees; // returns true
3 in trees; // returns true
6 in trees; // returns false
"bay" in trees; // returns false
// (you must specify the index number, not the value at that index)
"length" in trees; // returns true (length is an Array property)

// built-in objects
"PI" in Math; // returns true
const myString = new String("coral");
"length" in myString; // returns true

// Custom objects
const mycar = { make: "Honda", model: "Accord", year: 1998 };
"make" in mycar; // returns true
"model" in mycar; // returns true
```

#### instanceof

instanceof 用于判断某个对象是否是某个对象类型

```Javascript
objectName instanceof objectType
```

#### super

super 用于调用对象父级的属性，详细见 [super](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super)

### 解构赋值

```Javascript
// 数组解构
const foo = ["one", "two", "three"];
const [one, two, three] = foo; 
// 对象解构
const obj = {
    key1: 'val1',
    key2: 'val2',
    key3: 'val3'
}
const {key1, key2, key3} = obj
```

### 运算中的转化

- undefined 会转化为 NaN 或 false
- null 会转化为 0 或 false
- 数值与字符串  
由 字符串和数值组成的，仅由 **+** 链接的表达式，数值都会被自动转化为字符串，其余情况数值不会发生自动转化。

```Javascript
// '+' expression
console.log("I'm " + 18) // "I'm 18"
console.log(18 + ' is my age') // '18 is my age'
// other expression
console.log(18 - '9') // 9 
console.log(18 - '九') // NaN
```

可以用`parseInt(string, radix)`和`parseFloat(string)`函数将字符串分别转化为整型和浮点。

```Javascript
parseInt('18') // 18
// 以0~9开头的字符串仅被转化为开头字符组成的数值
parseInt('18is my age') // 18
parseInt('18<114514') // 18
// 若对浮点数值的字符串使用parseInt()会导致小数点精度丢失
parseInt('18.999is my age') // 18
parseFloat('18.999is my age') // 18.999
// parseInt(string, radix) 接收的第二个参数是进制数，默认为10
parseInt('111', 2) // 7
parseInt('1f', 16) // 31
```

还能用`BigInt()`方法将字符串或数值转为大整型。

```Javascript
BigInt(123) // 123n
BigInt("-12873263826387216387213") // -12873263826387216387213n
```

## 语句

这里着重记录分支语句与循环语句

### 分支语句

与C/Java写法一致，如下

```Javascript
// if 分支语句
if (condition1) {
  statement1;
} else if (condition2) {
  statement2;
} else if (conditionN) {
  statementN;
} else {
  statementLast;
}

// switch 分支语句
switch (expression) {
  case label1:
    statements1;
    break;
  case label2:
    statements2;
    break;
  // …
  default:
    statementsDefault;
}
```

这里着重记录Javascript中的可自动转化为false值的变量：

- false
- undefined
- null
- 0
- NaN
- "", 空字符串

### 意外处理语句

我们可以用throw语句或try...catch语句处理程序意外。  

- throw 语句
  什么数据类型都能被throw，但一般用Number和String来描述报错。
  已有的报错类型有：
  - [ECMAScript exceptions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error#error_types)
  - DOMException
  throw的语法就是`throw expression`
- try...catch 语句
  在try块中执行语句，若出现报错，控制流转到catch块进行报错处理,最后都会执行finally块中的内容，以下为一个例子

  ```Javascript
  openMyFile();
  try {
    writeMyFile(theData); // This may throw an error
  } catch (e) {
    console.error(e.name) // 报错名
    console.error(e.message) // 报错信息
    handleError(e); // If an error occurred, handle it
  } finally {
    closeMyFile(); // Always close the resource
    return false // finally中的返回作为整个语句的返回，能复写try/catch中的throw报错
  }
  ```

### 循环语句

与C/Java写法基本一致（但javascript循环中没有goto语句），如下：

```Javascript
// for 循环
for (initialization; condition; afterthought) {
      statement
}

// while 循环
while (condition){
  statement
}

// do while 循环
do {
  statement
}
while (condition);
```

但javascript有labeled语句，一般与break,continue配合使用：

```Javascript
foo: {
  console.log("face");
  break foo;
  console.log("this will not be executed");
}
console.log("swap");

// this will log:

// "face"
// "swap

/*******分割线********/
var allPass = true;
var i, j;

top: for (i = 0; items.length; i++)
  for (j = 0; j < tests.length; i++)
    if (!tests[j].pass(items[i])) {
      allPass = false;
      break top;
    }
```

### 迭代语句

迭代语句一般用于遍历数组和对象两种复杂数据类型。

```Javascript
// for ... in 迭代语句
for (const key in obj){
    console.log(`The key ${key} to the value in obj is ${obj[key]}`)
}

// for ... of 迭代语句
for (const item of arr){
    console.log(item)
}
// 或用Object.entries(obj)方法解构迭代对象
for (const [key, val] of Object.entries(obj)){
    console.log(key, val)
}
```

## 函数

### 函数声明

js中函数声明格式如下，

```Javascript
function funcName(parameter1, parameter2, ...){
    statement;
    return something;
}

funcName(p1, p2, ...)
```

因为函数的声明也会被提升，所以在函数声明前调用函数是没问题的：

```Javascript
console.log(add(1,1)) // 2
function add(a, b){
    return a+b;
}
```

<span style="color: #ff6d91">注意：</span> 函数表达式不会被提升，如下代码所示：

```Javascript
console.log(add(1,1)) //  ReferenceError
const add = function (a, b){
    return a+b;
}
```

### 函数作用域与闭包

函数内定义的变量为局部变量，无法在函数外直接得到，除非使用闭包。而函数内则访问局部变量和全局变量。

```Javascript
const v1 = 1;
function func1() {
    const v2 = 2;
    function closure() {
        const v3 = 3;  // 若这里将v3重命名为v2，func1内的v2将无法闭包
        return {v1, v2, v3}; // v1、v2、v3都能访问
    }
    // func1 内访问不到v3
    return closure
}
console.log(v1) // 1
console.log(v2, v3) // ReferenceError
const closure = func1()
console.log(closure()) // { v1: 1, v2: 2, v3: 3}
```

### arguments 对象

arguments对象是每个函数都有的存储所有传参的类数组-对象,示例如下：

```Javascript
const [pp, p1, p2, p3] = [0, 1, 2, 3]
function showArguments(default){
    console.log(arguments);
}
const params = [pp, p1, p2, p3] // 解构传参
showArguments(...params) // { 0: 0, 1: 1, 2: 2, 3: 3}
```

### 参数

#### 默认参数

```Javascript
function add(a, b = 1) {    // 默认参数一定写在非默认参数之后
    return a+b
}
console.log(add(1)) // 2
console.log(add(1,2)) // 3
```

#### 剩余参数

```Javascript
function showRestParams(param1, ...param2){
    console.log(param2)
}
showRestParams(1, 2, 3, 4) // [2, 3, 4]
```

## 内置对象

### Number & Math & Date

#### Number对象

内置的Number对象预置了一些常用属性和方法如下：

```Javascript
// Number 属性
const biggestNum = Number.MAX_VALUE;
const smallestNum = Number.MIN_VALUE;
const infiniteNum = Number.POSITIVE_INFINITY;
const negInfiniteNum = Number.NEGATIVE_INFINITY;
const notANum = Number.NaN;
// Number 方法
Number.parseFloat()
Number.parseInt()
Number.isFinite()
Number.isInteger()
Number.isNaN()
Number.isSafeInteger()
// Number 原型的方法
someNum.toExponential(precision) // 小数点后精确度
somNum.toFixed(precision) // 小数点后精确度
somNum.toPrecision(precision) // 有效数位
```

更多见 [详情](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number#static_properties)

#### Math对象

Math内置了很多数学常量和数学函数，一般需要时查表使用即可。

一般属性和方法有`Math.PI`,`Math.sin()`等。

更多见 [详情](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math#static_properties)。

#### Date对象

```Javascript
const newDate = Date() // 表示当下的时间的字符串
const newDate2 = new Date() // 创建当下时间的对象实例
const newDate3 = new Date('1999-9-9') // 创建1999-9-9T0:0:0:0的对象示例
const newDate4 = new Date(1995, 11, 31, 23, 59, 59, 999); // Set day and month

newDate.getFullYear() // 2024, 年份
newDate.getMonth() // 注意： getMonth()方法得到是月数-1，比如一月是0
newDate.getDate() // 仅日期
newDate.getTime() // 时间戳

newDate.parse(dateString) // 返回dateString对应的时间戳

newDate.set...() // get对应的set都有
```

### String

```Javascript
target.indexOf(pattern, position=0) // pattern在target中position之后的位置首次出现的位置
target.lastIndexOf(pattern)

target.startsWith(pattern, position=0)
target.endsWith(pattern, position=0)
target.includes(pattern, position=0)

str0.concat(str1, ..., strN) // 返回拼接后的字符串但不改变target
target.split(separator, limit) // 返回数组
target.slice(start, end = target.length) // 切片，start, end可以是负数
target.substring(start, end) // 负数作0处理，小数作起点，大数作终点

target.match(regex) // 返回一个数组，包括input,index,groups
target.matchAll(regex) // 返回一个迭代器，可用for of迭代， regex必需开启g全局，否则会报错

target.replace(regex/string, newString)
target.replaceAll(regex/string, newString)
target.search(regexp) // 返回下标，找不到返回-1

target.toLowerCase()
target.toUpperCase()

target.repear(count) // 重复拼接自身

target.trim() // 去除前后的空白
```

String的详细方法见[详情](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#static_methods)

还有一些和国际化标准有关的类：`Intl.DateTimeFormat`,`Intl.NumberFormat`,`Intl.Collator`

### Regex 正则表达式

#### 正则表达式字面量

```Javascript
// 正则表达式的字面量以 / content / 表示

// 边界符

// ^:开始
/^java/.test("javascript") // true
/^script/.test("javascript") // false
// $:结束
/java$/.test("javascript") // false
/script$/.test("javascript") // true

/^java$/.test("java") // true
/^java$/.test("javajava") // false

// 量词
// * >= 0次
/^java*$/.test('javaa') // true 
/^java*$/.test('jav') // true 
// + >= 1次
/^java+$/.test('java') // true
/^java+$/.test('') // false
// ？== 0次或1次
/^java?$/.test('java') // true
/^java?$/.test('') // true
// {n} == n次
/^java{1}$/.test('javajava') // false
/^java{2}$/.test('javajava') // true
// {n,} >= n次
/^java{1，}$/.test('javajava') // true
/^java{1,}$/.test('') // false 
// m次 >= {n, m} >= n次 

// 字符类
/[abc]/，出现其中一个字符就返回true 
/^[abc]$/.test('aa') false, 只能三个中的一个 
/^[abc]{2}$/.test('ab') true. 
/^[a-zA-Z0-9]$/ 表示所有大小写字母和数字中的一个 
/[^a-z]/除小写字母
.匹配除了换行符外的其他任何符号 
// 预定义：
\d， 相当于0-9 
\D, 相当于[^0-9] 
\w，任意字母数字下划线 
\W，[^\w]  
// 修饰符
/i 不区分大小写
/g 全局
```
