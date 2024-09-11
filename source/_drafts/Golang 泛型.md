

## Generics

Go 1.18之后开始支持泛型，以下是使用泛型的demo

```go
func getSum[T int|float32] (a, b T) {
    return a + b
}

// 使用泛型函数,可以显式指明需要的类型，或让编译器推理
getSum[int](1, 2)
getSum(1.2, 2.7)
```

### 类型约束

go 中可以声明类型约束，来代替对类型变量的类型指定。

```go
// 规则：同行内的元素用|连接表示并集，不同行间表示交集
type Nums interface {
    uint|int32|int64|float32
    int32|int64|float32|float64
}
// 等同于
type Nums2 interface {
    int32|int64|float32
}
```

go中提供了两种内置的类型约束：`comparable`和`any`。`any`是`interface{}`的别名，`comparable`表示所有用`!=`和`==`比较的类型

在类型约束中，还能通过`~`来表示对某一类型衍生类型的支持

```go
type MyInt int // MyInt为int类型的衍生类型，属于不同类型
type MyInt2 = int // MyInt2为int类型的别名，属于同一类型

type Int interface {
    ~int // 包括了MyInt
}
```

