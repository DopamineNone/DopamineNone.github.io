---
layout: passenge
title: 一些 Golang 的内置包
date: 2024-01-20 14:53:19
categories: Golang
tags: 
    - Golang
    - Grammar
---

## Reflect

> Go中每个**接口**变量都对应一个pair(value, concrete type), value是这个变量的值，而concrete type是这个变量在runtime系统中看见的类型。反射就是检查**接口**变量内部pair的机制

### Type & Value

reflect包提供了Type和Value两种核心类型，代表Go中的变量值和变量类型。可通过TypeOf和ValueOf方法获取变量的Type和Value。

reflect的TyepOf和ValueOf函数签名如下：

```Golang
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

这两函数先将传进来的变量转化为接口，再调用反射机制来实现对变量类型和值的查看

Type和Value也提供了一些方法：

- Type
  - Kind() 对应的底层类型,返回Kind(本质是uint)
  - Elem() 返回元素的类型Type,参数必须是array,chan,map,pointer,slice等
- Value
  - Kind()
  - Elem() 接口或指针对应的值
  - Type()
  - Interface() 以空接口的形式返回Value的值

### Field & Method

对于struct变量，可以通过NumField和NumMethod来遍历该结构体的字段和方法


```Golang
type AnoStruct struct {
	Name string
	Type string
}

Ano := AnoStruct{"Ano", "AnoType"}

anoType := reflect.TypeOf(Ano)
anoValue := reflect.ValueOf(Ano)

for i := 0; i < anoType.NumField(); i++ {
    field := anoType.Field(i)
    value := anoValue.Field(i)
    fmt.Println(field.Name, value)
}
```

### CanSet & Setxx

可以通过反射修改原变量

```Golang
// 必须传入指针配合Elem()方法来修改原变量的值
anoValue := reflect.ValueOf(&Ano).Elem()

if anoValue.CanSet() {
    anoValue.SetInt(100) // 还有SetInt/SetFloat/SetString等方法
}
```

### Call

可通过反射调用函数

```Golang
func (v* AnoStruct) DoSomething(val string) {
    fmt.Println(val)
}

...

anoFunc := anoValue.MethodByName("DoSomething")
args = []reflect.Value{reflect.ValueOf("Hello Reflect!")}
anoFunc.Call(args)
...
```

## Sort

掌握Sort包的一些函数可以方便刷lc（不是

### sort.Interface

```go
package sort
type Interface interface {
    Len() int            // 获取元素数量
    Less(i, j int) bool // i，j是序列元素的指数。
    Swap(i, j int)        // 交换元素
}
```



### sort.Sort

`sort.Sort`函数可对一个数组类型变量中的元素进行排序，且该数组类型变量需要实现`sort.Interface`接口。

```go
package demo/sort/Sort

import (
	"fmt"
    "sort"
)

type Person struct {
    Name string
    Age int
}

type PersonsByAge []Person

func (this Person)Len() {
    return len(this)
}

// 控制升序降序
func (this Person)Less(i, j int) {
    return demo[i].Age < demo[j].Age // 升序
}

func (this Person)Swap(i, j int) {
    this[i], this[j] = this[j], this[i]
}

func main() {
    demo := []Person{
        {"Jame", 55},
        {"Bob", 22},
        {"Alice", 34}
    }
    sort.Sort(PersonsByAge(demo))
    fmt.Println(demo)
    // {"Bob", 22},{"Alice", 34}, {"Jame", 55}
}
```

### sort.Slice

`sort.Sort()`的使用方法过于麻烦，相比下使用`sort.Slice()`方法能更轻松实现数组的自定义排序。其第一个参数就是待排序数组，第二个参数就是指明排序方法的`less`函数

```go
package demo/sort/Slice

import (
	"fmt"
    "sort"
)

type Person struct {
    Name string
    Age int
}

func main() {
    demo := []Person{
        {"Jame", 55},
        {"Bob", 22},
        {"Alice", 34}
    }
    sort.Slice(demo, func(i, j int) bool {
        return demo[i].Age < demo[j].Age
    })
    fmt.Println(demo)
    // {"Bob", 22},{"Alice", 34}, {"Jame", 55}
}
```

### sort.Ints / sort.Float64s

sort中有int和float数组的升序排序函数`sort.Ints(x []int)`和`sort.Float64(x []float64)`。

```go
package demo/sort/Ints

import (
	"fmt"
	"sort"
)

func main() {
	demo := []int{2,5,1,67,18}
	sort.Ints(demo)
	fmt.Println(demo)
}
```

### sort.Search

`sort.Search(n int, f func(i int) bool) int`用于返回$[0, n)$中最小的能够满足参数f的索引值。

```go
package demo/sort/Seach

import (
	"fmt"
	"sort"
)

func main() {
	demo := []int{2,5,1,67,18}
    x := 17
    // demo第一个大于等于x(17)的元素下标
    ans := sort.Search(len(data), func(i int) bool {
        return demo[i] >= x 
    })
	fmt.Println(ans, demo[ans], x)
}
```

`sort.SearchInts(x []int, target int)`,`sort.SearchFloat64s(x []float64, target float64)`和`sort.SearchStrings(x []string, target string)`作用与之类似，不展开描述。

## Container

container下包含了三个包：`list`、`heap`和`ring`，对应数据结构中的双向链表、堆、环形链表。

### List

实现源码：

```go
type Element struct {
   next, prev *Element // 前后指针
   list *List // 所属链表
   Value any // 值
}


type List struct {
   root Element // 哨兵元素
   len  int     // 链表元素个数
}
```



节点Element的Value的类型为空接口，故可以插入任意类型的数据。以下是**List的方法**

- 初始化：
  - `list.New() *List`
- 增: 
  - `(s *list)PushFront(x any)`
  - `(s *list)PushBack(x any)`
  - `(s *list)PushFrontList(other *List)`
  - `(s *list)PushBackList(other *List)`
  - `(s *list)InsertAfter(v any, mark *Element)`
  - `(s *list)InsertBefore(v any, mark *Element)`
- 删
  - `(s *list)Remove(e *Element) any`, 返回被删除节点的值
  - `(s *list)Init() *List`, 清空列表
- 改
  - `(s *list)MoveToFront(e *Element)`
  - `(s *list)MoveToBack(e *Element)`
  - `(s *list)MoveBefore(e, mark *Element)`
  - `(s *list)MoveAfter(e, mark *Element)`
- 查
  - `(s *list)Back() *Element`
  - `(s *list)Front() *Element`
  - `(s *list)Len() int` O(1)



可通过**Element**的`Next() *Element`和`Prev() *Element`方法进行节点操作

### heap

和sort包一样，heap也有一个Interface类型。

```go
type Interface interface {
    sort.Interface
    Push(x interface{}) // 向末尾添加元素
    Pop() interface{}   // 从末尾删除元素
}
```

所有实现该接口的类型也能用`container/heap`包提供函数构建最大堆/最小堆（由sort.Interface的Swap函数决定）。

常用的**heap包提供的函数**有：

- 初始化
  - `Init()`, 即buildHeap()
- 增
  - `Push(h Interface, v any)`
- 删
  - `Pop(h Interface)`
  - `Remove(h Interface, i int)`， 删除下标为i的元素
- 改
  - `Fix(h Interface, i int)`，若下标i处被修改，可使用Fix函数创新从i处构造堆

### ring

实现源码：

```go
type Ring struct {
   next, prev *Ring // 前后指针
   Value      any   // 值，使用者自己设置
}
```

ring的一些方法

- 创建：
  - `New(i int)`，i为节点个数
- 增：
  - `(s *Ring)Link(r *Ring) *Ring`, 当前环s与r拼接,使s的下个元素为r，并返回原本s的下个元素
- 删：
  - `(s *Ring)Unlink(n int) *Ring`，删除从s.Next()开始的`n % s.Len()`个元素，返回被删除的环
- 查：
  - `(s *Ring)Next() *Ring`
  - `(s *Ring)Prev() *Ring`
  - `(s *Ring)Len() int`
- 其他：
  - `(s *Ring)Do(f func(a any))`，对环的每个元素调用f函数
  - `(s *Ring)Move(n int) *Ring`，s移动`|n|%s.Len()`个位置，`Move(1)==Next()`，`Move(-1)==Prev()`

## Buffer

Golang标准库下的缓冲区，可存储字节。

### 声明

```Go
var buffer bytes.Buffer
buffer := new(bytes.Buffer)
buffer := bytes.NewBuffer(s []byte)
buffer := bytes.NewBufferString(s string)
```

### 写入

```Go
buffer.Write(s []byte) (n int, err error)
buffer.WriteString(s string) (n int, err error)
buffer.WriteByte(b byte) err error
buffer.WriteRune(r rune) (n int, err error)
buffer.ReadFrom(f io.Reader) (n int, err error)
```

### 读出

```golang
buffer.Read(p []byte) (n int, err error)
buffer.Next(n int) []byte
buffer.ReadByte() (byte, error)
buffer.ReadBytes(delimiter byte) (line []byte, err error)
buffer.ReadString(delimiter byte) (line string, err error)
buffer.WriteTo(w io.Writer) (n int64, err error)
```

## Regexp

正则表达式

### 构建正则对象

```go
reg := regexp.MustCompile(s string) // s为正则表达式
```

### 判定字符串是否符合正则表达式

```go
flag, err := regex.MatchString(reg string, s string)
flag, err := reg.MatchString(s string) // reg为正则对象*Regexp
```
