---
layout: passenge
title: Golang 学习笔记
date: 2024-01-20 14:53:19
categories: Golang
tags: 
    - Golang
    - Grammar
---

<!-- Golang 知识清单 -->
<!-- 这篇博客仅是本人在学习Golang语法基础时的一些记录。 -->

## 常量

- 关键词： `const`
- 用法：
  1. 单行声明：`const variableName [Type] = value`
  2. 并行声明：`const p1, p2, p3 = v1, v2, v3`
  3. 多行声明：

  ```Go
  const beef, two, c = "eat", 2, "veg"
  const Monday, Tuesday, Wednesday, Thursday, Friday, Saturday = 1, 2, 3, 4, 5, 6
  const (
      Monday, Tuesday, Wednesday = 1, 2, 3
      Thursday, Friday, Saturday = 4, 5, 6
  )
  const (
      a = iota
      b = iota
      c = iota
  )
  ```

## 变量

### 声明格式

单行变量声明格式`var name [type] [= val]`

多行变量声明格式:

```Go
var (
    a int
    b bool
    str string
)
```

也可以同时给多个值声明类型：`var a, b, c int`

### 初始化

若未显示初始赋值，则：

- int: 0
- float: 0.0
- string: 空字符串“”
- bool: false
- ptr: nil

且声明时变量未声明类型，则编译器会通过初始赋值推导变量赋值；当然，未声明类型的变量必需得初始赋值。

```Go
// Right
var userName = "Joe" // Go变量命名遵循小驼峰命名法
var lottery int64 = 100
var temp float64

// Wrong
var unknow
```

### 值类型和引用类型

学过C的都懂。

Go中通过`&`运算符来得到变量的地址。

### 打印

`Printf()`函数可以在fmt包外使用，可以向控制台打印格式化字符串，同C语言的`printf()`和Go的`fmt.Sprintf()`。

`fmt.Print()`和`fmt.Println()`的作用一致，使用%v对字符串进行格式化，`fmt.Println()`会多打印一个\n。

### := 初始赋值运算符

可以使用`:=`来高效地进行变量声明和初始化。

`var num = 10`就可以简写为`num := 10`。

**注意**： 对同一个变量只能使用一次`:=`运算符，否则会报错；对已声明但未使用的局部变量也会出现报错；在变量声明前使用变量也会报错。

### 变量类型

- bool
- int/uint/uintptr
  - int/uint 长度与操作系统位数相等, int16/int32/int64间不能隐式转化，常量除外。
  - uintptr 长度足够存放一个指针。
- float32/float64 **(没有float和double!!! 尽量用float64)**
- byte (int8的别名，字符)， 自行了解utf8包。
- string **(Go中字符串没有以'\0')**, 自行了解strings和strconv包。
- ptr **(Go中指针运算是非法的)**

### 变量运算

运算符同C语言；唯一需注意的是带有`++`、`--`运算符只能当作语句，像`sum = i++`这种语句在Go中不合法。

符号优先级优先级     运算符

```Text
 7      ^ !
 6      * / % << >> & &^
 5      + - | ^
 4      == != < <= >= >
 3      <-
 2      &&
 1      ||
 ```

## 控制结构

### if-else 结构

```Go
if condition1 {
    // do something 
} else if condition2 {
    // do something else    
} else {
    // catch-all or default
}

if val := 10; val > max {
    // do something
}
```

**注意**：

1. 花括号\{\}无论如何都不能省略
2. \{与关键字同一行，\}也与关键字同一行
3. 条件中可包含初始化语句

### switch 结构

```Go
// switch 的第一种形式

switch i {
    case 0: // 空分支，只有当 i == 0 时才会进入分支
    case 1:
        f() // 当 i == 0 时函数不会被调用
    default: 
        DoSomething()
}

switch i {
    case 0: fallthrough
    case 1:
        f() // 当 i == 0 时函数也会被调用
}

// switch 的第二种形式
switch {
    case condition1:
        ...
    case condition2:
        ...
    default:
        ...
}

//  switch 的第三种形式
switch initialization {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
```

### for 结构

1. 基于计数器的for语句结构：`for 初始化语句; 条件语句; 修饰语句 {}`
2. 基于条件判断的for语句:`for 条件语句 {}`
3. 无限循环: `for {}`
4. for-range结构: `for ix, val := range coll { }`， **注意**val只是对coll中值的拷贝

### goto 结构

配合标签使用，不建议使用goto语句

## 函数

### 介绍

Go中一共有三种函数：

1. 具名函数
2. 匿名函数 (Lambda)
3. 方法

而且Go允许一个函数A作为另个函数B的参数传入，只要A的返回值数量和类型与B函数参数一致

Go 禁止函数重载

Go中申明一个在外部定义的函数，你只需要给出函数名与函数签名，不需要给出函数体，如`func flushICache(begin, end uintptr)`

### 参数和返回值

#### 传参

传参有两种方式：

1. 按值传参
2. 引用传参

推荐使用引用传参，一般比按值传参有更小的性能开支

具有多个参数的函数也能直接传入一个包含多个变量的slice作为参数

#### 返回值

返回值有一些特性

1. 可命名可匿名，使用多个匿名返回值或单个及其以上的命名返回值时需要用()括起来
2. 使用命名返回值则return语句可不带参数

```Go
package main

import "fmt"

var num int = 10
var numx2, numx3 int

func main() {
    numx2, numx3 = getX2AndX3(num)
    PrintValues()
    numx2, numx3 = getX2AndX3_2(num)
    PrintValues()
}

func PrintValues() {
    fmt.Printf("num = %d, 2x num = %d, 3x num = %d\n", num, numx2, numx3)
}

func getX2AndX3(input int) (int, int) {
    return 2 * input, 3 * input
}

func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```

#### 空白符

空白符`_`，用于匹配不需要的函数返回值

#### 变长参数

变长参数类型用`...type`表示，如`func myFunc(a, b, arg ...int) {}`，该类型和slice很像，可用for迭代。

#### defer与追踪

defer 关键字允许我们在函数返回返回值前才执行某些语句。defer 与 return之间执行顺序是： 先为返回值赋值，再执行defer，最后返回返回值。

```Go
package main
import "fmt"

func main() {
    function1()
}

func function1() {
    fmt.Printf("In function1 at the top\n")
    defer function2()
    fmt.Printf("In function1 at the bottom!\n")
}

func function2() {
    fmt.Printf("function2: Deferred until the end of the calling function!")
}

// Print Results
// In Function1 at the top
// In Function1 at the bottom!
// Function2: Deferred until the end of the calling function!
```

当有多个 defer 行为被注册时，它们会以逆序执行（类似栈，即后进先出）

这样我们就能通过defer关键字实现代码追踪，如

```Go
package main

import "fmt"

func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

func a() {
    trace("a")
    defer untrace("a")
    fmt.Println("in a")
}

func b() {
    trace("b")
    defer untrace("b")
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

### 内置函数

详情见 [内置函数](https://learnku.com/docs/the-way-to-go/built-in-function/3603)

### 递归函数

基本与C语言一致。Go语言中允许相互调用的递归函数，这些函数的声明顺序是随意的。

### 闭包

这里放个例子：

```Go
package main

import "fmt"

func main() {
    // make an Add2 function, give it a name p2, and call it:
    p2 := Add2()
    fmt.Printf("Call Add2 for 3 gives: %v\n", p2(3))
    // make a special Adder function, a gets value 2:
    TwoAdder := Adder(2)
    fmt.Printf("The result is: %v\n", TwoAdder(3))
}

func Add2() func(b int) int {
    return func(b int) int {
        return b + 2
    }
}

func Adder(a int) func(b int) int {
    return func(b int) int {
        return a + b
    }
}
```

## 数组

### 声明

#### 数组声明格式

```Go
var identifier [len]type
```

像让数组接收任意类型的元素需要使用空接口作为类型

Go 中数组是值类型（不是指针），故可通过`new()`创建，如`new([5]int)`。

通过`var`和`new()`分别创建的数组arr1和arr2的区别是:

- arr1类型是*[5]int
- arr2类型是[5]int

#### 数组常量

有三种形式：

- `[5]int{1,2,3,5,6}`
- `[...]int{1,2,3,4,6}`
- `[5]int{3: 4, 5: 6}`

#### 多维数组

`[4][4]int`

#### 数组传递给函数

`func demo(a *[3]int){}`

#### 数组相等

数组相等判断规则：

```Golang
v1 := [3]string{"Golang", "Python", "C++"}
v2 := [3]string{"Golang", "Python", "C++"}
v3 := [3]string{"Golang", "C++", "Python"}
v4 := [3]string{"Golang", "Python"}
v5 := []string{"Golang", "Python", "C++"}
v6 := []string{"Golang", "Python", "C++"}

fmt.Println(v1 == v2) // true
fmt.Println(v1 == v3) // false 元素顺序不同
fmt.Println(v1 == v4) // false 元素个数不同
// fmt.Println(v1 == v5) mismatch types: [3]string and []string
// fmt.Println(v5 == v6) slice can only be compared with nil
// fmt.Println(v3 == v5) mismatch types: [3]string and []string
```

### 切片

>切片（slice）是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型（因此更类似于 C/C++ 中的数组类型，或者 Python 中的 list 类型）。

```Go
// Equal Expression
slice[:] == slice
slice[:n] == slice[0:n]
slice[n:] == slice[n:len(slice)]
```

#### cap() --数组容量

切片是长度可变的数组， `0 <= len(s) <= cap(s)`，其中`cap()`用于返回s切片的容量。多个相关的切片是共享数据的。

#### 切片传递给函数

`func demo(a []int){}`

#### make() 创建切片

`var slice []type = make([]type, len, [cap])`

new和make的区别：

- new是分配新的空间，返回的是指针
- make是返回初始值，只使用于数组，map和channel

#### bytes 包

提供操作`[]byte`类型方法的包

#### for-range

for-range句式用于遍历slice，其中ix为索引，value为索引值。

```Go
for ix, value := range slice1 {
    ...
}
```

#### 复制与追加

```Go
slice1 := make([]int, 10)
slice2 := []int{1,2,3}

n := copy(slice1, slice2)
fmt.Print(n, "\n")
fmt.Print(slice1)

slice2 = append(slice2, 4, 5, 6)
fmt.Print(slice2)
```


#### sort 包

Go提供了sort包来实现各种切片的排序，如`sort.Ints(a []int)`, `sort.Strings(s string)`

## Map

### Map 声明格式

Map的声明格式为`var m map[keytype]valuetype`

keytype 只能为简单类型如`int`,`string`, `float`等，切片和结构不能作为keytype,未初始化的值为`nil`。

用`make()`初始化map变量，而不是用`new()`

```Go
package main
import "fmt"

func main() {
    mf := map[int]func() int{
        1: func() int { return 10 },
        2: func() int { return 20 },
        5: func() int { return 50 },
    }
    fmt.Println(mf)
}
```

### 查键和删键

用`val, isPresent = m["key"]`中的`isPresent`可以判断键值`key`是否存在。

用`delete(m, key)`可以删键，且key不存在也不会报错

### Map与for-range

```Go
for key, value := range m {
    ...
}
```

## Package

### 标准库

如`fmt`,`os`常用功能的150+内置包称作标准库。常见标准库看[这里](https://learnku.com/docs/the-way-to-go/overview-of-the-91-standard-library/3626)

### regexp

- 有`Match`方法：`ok, _ := regexp.Match(pat, searchIn)`

- 同样有`MatchString`方法

- 创建正则对象：`re, _ = regexp.Compile(pat)`

- 替换字符串： `str := re.ReplaceAllString(pat, new)`

- 按函数替换字符串： `str := re.ReplaceAllStringFunc(pat, f)`

### 锁和sync

为避免同一变量同时被不同线程访问修改造成资源竞争，我们需要对变量上一个线程锁

```Go
import "sync"

func main() {
    var mu sync.Mutex
    var count int = 0
    for i := 0; i < 10000; i++ {
        mu.Lock() // 加锁，保护count
        count++
        mu.Unlock() // 解锁
    }
}
```

还有`RWMutex`锁，使用`RLock()`来允许同时有多个线程对变量进行读操作，只有一个线程进行写操作

### 精密计算big包

Go提供了`big`包来进行精确计算。

- `big.NewInt(n)`
- `big.NewRat(N, D)` N为分子，D为分母

### 自定义包

包文件名应由短小的不含`_`的单词组成

想使用包得先通过`import`关键字导入包： `import "relative path or URL"`

- `import "package"`
  
  使用包的全局变量和函数需通过`包名.val`和`包名.func`

- `import . "package"`

  可直接使用包的全局变量和函数

- `import _ "package"`

  只导入包的副作用，即调用init函数和初始化全局变量

- `import alias "package"`
  
  导入包并使用`alias`别名

格式如下：

```Go
// pack_demo包
package packdemo

import "fmt"

var pi float32 = 3.14514

func HelloWorld() {
    fmt.Println("Hello World")
}
```

```Go
// 主函数入口
package main

import (
    "fmt"
    "./pack_demo" // 同一路径下
)

func main() {
    fmt.Println(packdemo.pi)
    packdemo.HelloWorld()
}
```

## 结构与方法

### 结构体定义

```Go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

声明结构体变量

```Go
var s T
s.a = 5
s.b = 8

// use new()
var t *T
t = new(T)
```

值得注意的是，Go中无论是结构体变量还算结构体指针都通过`.`选择器符来引用结构体字段。

更简短的初始化结构体实例为

```Go
demo := &struct1{10, 1.5, "no"} // 本质是使用new()
// or
var demo struct1
demo = struct1{10, 1.5, "no"}
```

Go中结构体的内存分布是连续块存在的。

结构体中的转化只存在于有相同底层类型的结构体间，而且得通过显式转化。

### 工厂方法

```Go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}

// 调用工厂方法创建结构体实例
f := NewFile(10, "./test.txt") 
```

在Go中，标识符以大写字符开头的能被外部包的代码所引用，否则对外部包不可见。通过这个能使一个结构体设置为私有类型，只能通过工厂方法创建和操作结构体。

通过`make()`创建结构体会报错。

### 结构体中的标签

用于自省类型、文档标记等。可通过`reflect`包来获取。

```Go
package main

import (
    "fmt"
    "reflect"
)

type TagType struct { // tags
    field1 bool   "An important answer"
    field2 string "The name of the thing"
    field3 int    "How much there are"
}

func main() {
    tt := TagType{true, "Barak Obama", 1}
    for i := 0; i < 3; i++ {
        refTag(tt, i)
    }
}

func refTag(tt TagType, ix int) {
    ttType := reflect.TypeOf(tt)
    ixField := ttType.Field(ix)
    fmt.Printf("%v\n", ixField.Tag)
}
```

### 匿名字段和内嵌结构体

匿名字段可用于模拟结构体的继承，即通过内嵌匿名结构体来模拟。

```Go
package main

import "fmt"

type innerS struct {
    in1 int
    in2 int
}

type outerS struct {
    b    int
    c    float32
    int  // anonymous field
    innerS //anonymous field
}

func main() {
    outer := new(outerS)
    outer.b = 6
    outer.c = 7.5
    outer.int = 60
    outer.in1 = 5
    outer.in2 = 10

    fmt.Printf("outer.b is: %d\n", outer.b)
    fmt.Printf("outer.c is: %f\n", outer.c)
    fmt.Printf("outer.int is: %d\n", outer.int)
    fmt.Printf("outer.in1 is: %d\n", outer.in1)
    fmt.Printf("outer.in2 is: %d\n", outer.in2)

    // 使用结构体字面量
    outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
    fmt.Println("outer2 is:", outer2)
}
```

命名冲突时：

- 外层名字会覆盖内层名字
- 同一级别出现两次相同的名字会报错

### 方法

Go 中方法是作用在receiver上的函数。定义格式如下：

```Go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

无论recv是实例还是指针，都通过`object.funcName()`调用方法。

不需要recv的值可以用`_`代替：`func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }`

值得注意的是类型和作用其上的方法必需在同一包中，这也是为什么不能在int,float等类型上定义方法。

结构体内嵌和自己在同一个包中的结构体时，可以彼此访问对方所有的字段和方法。

### 类型的String()方法

为结构体定义`String()`方法可以使`fmt.Printf()`的%v输出或`fmt.Print()`和`fmt.Println()`的默认输出

### 垃圾回收和SetFinalizer

`runtime.GC()`可以显式调用垃圾收集器

## 接口

### 接口定义

- 类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口。

- 实现某个接口的类型（除了实现接口方法外）可以有其他的方法。

- 一个类型可以实现多个接口。

- 接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）

```Go
package main

import "fmt"

type stockPosition struct {
    ticker     string
    sharePrice float32
    count      float32
}

/* method to determine the value of a stock position */
func (s stockPosition) getValue() float32 {
    return s.sharePrice * s.count
}

type car struct {
    make  string
    model string
    price float32
}

/* method to determine the value of a car */
func (c car) getValue() float32 {
    return c.price
}

/* contract that defines different things that have value */
type valuable interface {
    getValue() float32
}

func showValue(asset valuable) {
    fmt.Printf("Value of the asset is %f\n", asset.getValue())
}

func main() {
    var o valuable = stockPosition{"GOOG", 577.20, 4}
    showValue(o)
    o = car{"BMW", "M3", 66500}
    showValue(o)
}
```

### 类型断言

检查断言的方法：

```Go
// varI 是接口类型变量

// uncheck type
v := varI.(T) // 若varI含类型T的值，则v为varI转化为T的值，否则v为T类的零值

// best
if v, ok := varI.(T); ok {
    Process(v)
    return
}
```

也可以用Go中特有的`type-switch`语句来判断：

```Go
switch t := varI.(type) { // 这里的type画重点！！
    case type1:
        // do something
    case type2: 
        ...
    default:
        ...
}
```

具体类型是值类型还算引用类型取决于该类的接口实现使用的接收者类型（雾

判断某个值是否实现某个接口也可以用`varT.(I)`的方法来判断，这里不多赘述

Go 语言规范定义了接口方法集的调用规则：

- 类型 \*T 的可调用方法集包含接受者为 \*T 或 T 的所有方法集
- 类型 T 的可调用方法集包含接受者为 T 的所有方法
- 类型 T 的可调用方法集不包含接受者为 \*T 的方法

### 空接口

任何类型都实现了空接口，所以可以这样做：

```Golang
type Any interface {}

type Person struct {
    name string
    age int
}
var any Any
any = "ABC"
fmt.Println(any)
any = 15
fmt.Println(any)
any = 1.05
fmt.Println(any)
any = new(Person{name: "Bob", age: 100})
fmt.Println(any.name, " ", any.age)
```

将切片数据复制到空接口切片只能通过`for-range`语句显式赋值

## 读写数据

### 读取用户输入

- `fmt.Scanln(&str1, &str2, ...)`从标准输入依次读取以空格分割的字符串，直到遇到`\n`
- `fmt.Scanf(format, &str1)`
- `fmt.Sscan(str, &str1, &str2, ...)` 从字符串读取

还能用bufio包提供的缓冲读取来实现读取输入。

```Go
import (
    "fmt"
    "os"
    "bufio"
)
inputReader := bufio.NewReader(os.Stdin)
input, err := inputReader.ReadString('\n')
```

### 文件读写

通过`os`包的`Open()`函数能得到文件的句柄

```Go
// import ("os", "bufio")
inputFile, inputError := os.Open("path/to/file")
if inputError != nil {
    // Report FileNotFound Error
    return
}
// Remember close the file in the end!!!
defer inputFile.Close()

// Read the file
inputReader := bufio.NewReader(inputFile)

for { // Infinite loop until EOF
    inputStr, readerError := inputReader.ReaderString('\n')
    if readerError == io.EOF {
        return 
    }
}
```

还可以用`io/ioutil`将整个文件内容读到一个字符串（`[]byte`）中,

```Go
// import ("io/ioutil")

inputFile, err := ioutil.ReadFiler("path/to/file")
if err != nil {
    // Error Handling
}

err = ioutil.WriteFile("path/to/file", buf, max_length) // 可命名文件名
if err != nil {
    // Error Handling
}
```

带缓冲的读取

```Go
buf := make([]byte, 1024)

n, err := inputReader.Read(buf) // n为读到的字节数
if n == 0 {break}
```

`Fscanln`读取文件数据

```Go
// inputFile --- file handler
_, err := fmt.Fscanln(file, &v1, &v2, &v3)
```

切片读文件

```Go
// n是字符数， f是文件句柄， buf是读取到文件内容的切片
n, err := f.Read(buf[:])
```

写文件：

```Go
file, err := os.Open("data.txt", os.O_WRONLY|os.O_CREATE, 1024)
if outputError != nil {
    fmt.Printf("An error occurred with file opening or creation\n")
    return  
}
defer outputFile.Close()

outputWriter := bufio.NewWriter(outputFile)
outputString := "hello world!\n"

for i:=0; i<10; i++ {
    outputWriter.WriteString(outputString)
}
outputWriter.Flush() // 重点！！！
```

使用`io`包的`Copy(dst, src)`函数可以实现文件的拷贝，其中dst和src是文件句柄。

### 命令行参数

通过`os.Args`可以获取命令行参数的切片(`os.Args[0]`是程序名)

`flag`包也能处理命令行参数：

```Go
import (
    "fmt"
    "os"
    "flag"
)

// flag.Bool(flag, bool, demonstration)
var isN *bool = flag.Bool("n", false, "default demonstrations")
for i := 0; i < flag.NArgs(); i++ {
    fmt.Println(flag.Arg(i))
    if
}
```

### JSON数据

Go中用`encoding/json`包来处理json数据

Go结构体 => JSON

```Go
import (
    "encoding/json"
    "fmt"
)

type Adr struct {
    Country  string
    Province string
    Code     int
}

type Dic struct {
    Name    string
    Age     int
    Address *Adr
}

func ShowJson(demo *Dic) {
    json, _ := json.Marshal(demo) // 序列化
    fmt.Println(json)
    var res *Dic
    _ := json.Unmarshal(json, res)
    fmt.Println(res)
}

func main() {
    demo := &Dic{Name: "Dopa", Age: 18, Address: &Adr{"China", "Fujian", 200}}
    ShowJson(demo)
}
// {"Name":"Dopa","Age":18,"Address":{"Country":"China","Province":"Fujian","Code":200}}
```

Go中与JSON对应的数据结构：

- bool 对应 JSON 的 booleans
- float64 对应 JSON 的 numbers
- string 对应 JSON 的 strings
- nil 对应 JSON 的 null

编码Map对象需要是map[string] T类型

Channel,复杂类型不能被编码

map[string]interface{}和[]interface{}能解码任何JSON对象和数组

## 错误处理

### 定义错误

Go中提供了`errors`包来定义错误

```Go
import (
    "fmt"
    "errors"
)

var NotFoundErr error = errors.New("File not found")

func main() {
    fmt.Println(NotFoundErr)
}
```

也可以用`fmt.Errorf()`创建错误对象

```Go
err := fmt.Errorf("usage: %s", something)
```

### 运行时异常

Go中用`panic`产生运行时异常

```Go
func ReadInfo() {
    file, err := os.Open("error.txt")
    if err != nil {
        panic("Fail to Open File")
    }
}
```

### 从panic恢复

在多次嵌套调用的函数中触发panic使defer 语句保证执行并且控制权交给panic调用的函数。栈会被展开直到defer语句中的recover()被调用

```Go
func protect(g func()) {
    defer func() {
        if err := recover(); err != nil {
            log.Printf("run time err : %v", err)
        }
    }()
    log.Println("start")
    g()
}
```

所以我们就能用闭包处理错误

```Go
import (
    "fmt"
    "error"
)

func ConvertInt64ToInt(input int64) int {
    if math.MinInt32 <= input && input <= math.MaxInt32 {
        return int(input)
    }
    panic(fmt.Sprintf("%d is out of the int32 range", input))
}

func IntFromInt64(input int64) (int, error) {
    var err error = nil
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("%v", r)
        }
    }()
    return ConvertInt64ToInt(input), err
}
```

## 测试

### 单元测试和基准测试

测试代码的包文件名满足这种形式`*_test.go`，且必需导入`testing`包，写一些Test*开头的全局函数

```Go
func TestAbcde(t *testing.T)
```

一些二通知测试失败的函数：

```Go
func (t *T) Fail() // 标记测试函数为失败，然后继续执行（剩下的测试）。
func (t *T) FailNow() // 标记测试函数为失败并中止执行；文件中别的测试也被略过，继续执行下一个文件。
func (t *T) Log(args ...interface{}) // args 被用默认的格式格式化并打印到错误日志中。
func (t *T) Fatal(args ...interface{}) //     结合 先执行Log，然后执行FailNow的效果。
```

使用`go test`来编译测试程序，并执行所有Test\*的函数，所有函数通过会打印PASS

做简单的基准测试需要测试代码中包含Benchmark*的函数并接收一个*testing.B类型的参数。

```Go
func BenchmarkReverse(b *testing.B) {
    ...
}
```

命令 go test –test.bench=.* 会运行所有的基准测试函数；代码中的函数会被调用 N 次（N 是非常大的数，如 N = 1000000），并展示 N 的值和函数执行的平均时间，单位为 ns（纳秒，ns/op）。

### 表驱动测试

```Go
var tests = []struct{   // Test table
    in  string
    out string

}{
    {“in1”, “exp1”},
    {“in2”, “exp2”},
    {“in3”, “exp3”},
...
}

func TestFunction(t *testing.T) {
    for i, tt := range tests {
        s := FuncToBeTested(tt.in)
        if s != tt.out {
            t.Errorf(“%d. %q => %q, wanted: %q”, i, tt.in, s, tt.out)
        }
    }
}
```

## 协程与通道

### 并发、并行和协程

1. 一个应用程序是运行在机器上的一个进程，进程是一个运行在自己内存地址空间里的独立执行体。
2. 一个进程由一个或多个操作系统线程组成，线程其实是共享同一个内存地址空间的一起工作的执行体。
3. 协程运行在线程之上，协程并没有增加线程数量，只是在线程的基础之上通过分时复用的方式运行多个协程。
4. Go使用`go`关键字就能开启协程，要注意若主线程先结束，未结束的协程也会中断。

### GOMAXPROCS

用GOMAXPROCS 为一个大于默认值 1 的数值来允许运行时支持使用多于 1 个的操作系统线程，否则所有的协程都会共享同一个线程。

协程的数量 > 1 + GOMAXPROCS > 1。

```Go
runtime.GOMAXPROCS(2)
```

### channel

channel类型本质是队列。声明格式为`var identifier chan datatype`，之后必需实例化它：`identifer = make(chan datatype)`

用`var identifier <- chan datatype`声明只读通道， `var identifier chan <- datatype`声明只写通道。

通道操作符为`<-`:

```Go
ch := make(chan int)
ch <- 100 // 向ch写入100
fmt.Println(<- ch) // 100
fmt.Println(len(ch)) // 0; 100被取出
```

> 通道阻塞： 当通道数据已满且没有接收者读出数据/通道已空仍有接收者尝试读出数据，就会触发通道阻塞，直到数据被读出/通道读入新数据

遍历channel:

```Go
ch := make(chan int, 100)

...
close(ch) // 阻塞管道后才能遍历

// 方法一 自动检测管道是否被阻塞
for val := range ch {
    dosomethings()   
}
// 方法二 用ok判断管道是否被阻塞
for {
    val, ok := <- ch
    if !ok {
        break
    }
    dosomethings()
}
// 错误的方法！
for i := 0; i < len(ch); i++ {
    // len(ch)会变化！
}
```

可以用select语句切换协程

```Go
select {
case u:= <- ch1:
        ...
case v:= <- ch2:
        ...
        ...
default: // no value ready to be received
        ...
}
// 如果都阻塞了，会等待直到其中一个可以处理
// 如果多个可以处理，随机选择一个
// 如果没有通道操作可以处理并且写了 default 语句，它就会执行：default 永远是可运行的（这就是准备好了，可以执行）。
```

### sync包的WaitGroup

WaitGroup 用于创建任务队列，其中提供了三个方法`WaitGroup.Add(count)`、`WaitGroup.Done()`、`WaitGroup.Wait()`

```Go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var waitGroup sync.WaitGroup
    waitGroup.Add(1)
    go func() {
        defer waitGroup.Done()
        doSomethings()
    }()
    waitGroup.Wait()
    fmt.Println("main end!")
}
```
