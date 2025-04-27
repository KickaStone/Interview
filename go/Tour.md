# A tour of go

https://tour.go-zh.org/list

## 包，变量和函数


### 变量导出
**多行导入**
```go
import "fmt"
import "math"
```

也可以写在一个括号里，更集中
```go
import (
    "fmt"
    "math"
)
```

**导入外部包**
以上都是引入go的标准库，go去`GOROOT`下加载模块，如果引入自己写的模块，可以使用如下方式
```go
import "./packageFolder" // 相对路径, 不建议使用
import "path/to/package" // 使用$GOPATH/src/path/to/package
```

第二种是通用安装第三方库的方式，通常还需要下载并安装第三方库到$GOPATH中。
`go get github.com/packagePath`

**导入操作**

1. 点导入

    `import . "fmt"`
    调用这个包的函数时可以省略包名（不推荐）

2. 别名
    `import (f "fmt")`
    相当于把包名替换成了设置的名称

3. _ 操作
    `import (_ "package")`
    只是使用包的`init()`函数，（导入包时，`init`函数自动执行），无法使用包的导出函数



### 导出？为什么要大写

如果一个名字以大写字母开头，它就是已导出的。例如，Pizza就是个已导出名, Pi也同样，它导出自math包。
导入一个包时，只能使用其中已导出的内容，未导出的名字在包外均无法访问。


### 为什么用变量类型在后面的语法声明？

团队一篇blog写了相关设计思想
https://blog.go-zh.org/gos-declaration-syntax



### 函数的多返回值

错误用法❌

```go
func swap(a, b int) (b, a int) {}
```

在 Go 语言中，带有命名返回值的函数允许你用返回值的名字来直接返回值，但是该名字不能与参数名字相同。这是因为在 Go 的命名返回值中，返回值的名字在函数体中是独立的变量，名称的重定义会导致歧义。

正确的用法✅

```go
func swap(a, b int) (y, x int){
  y = b
  x = a
}
```



### 变量声明

函数外的每个语句都**必须**以关键字开始(`var, func`)等，因此`:=`结构不能在函数外使用



### 基本类型

列举以备查询

`int,uint`和`uintptr`默认和机器位宽一致。

```go
bool

string

int int8 int16 int32 int64
uint uint8 uint16 uint32 uint64 uintptr

byte // unint8别名

rune // int32别名, 表示Unicode码

float32 float64

complex64 complex128
```

### 类型转换

go中任何类型转换都必须显示声明，不支持隐式类型转换，为了避免发生错误。

类型在声明时可以指明，也可以通过初始化的值自动推断。如果不指明类型，新变量类型可能是`int, float64, complex128`取决于精度。



### 常量

使用`const`关键字声明常量。



## 控制流语句 `for, if, else, switch，defer`



### `for`循环

go只有`for`一种循环语句，但可以实现很多循环类型

```go
for {
  //... 无限循环
}

// while循环
for sum < 1000 {
  sum += a;
}

// 常规循环 init; cond; post
for i := 0; i < 10; i++ {
  sum += 1
}
// init: 第一次迭代前执行
// cond: 每次迭代前求值，若为false，循环迭代就会终止
// post: 每次迭代的结尾执行
```



```go
// for len 遍历元素
a := [5]int{1, 2, 3, 4, 5}
for i := 0; i < len(a); i++{
  fmt.Println(a[i])
}

// for range 遍历元素
a := [5]int{1, 2, 3, 4, 5}
for i := range a {
  fmt.Println(a[i])
}
for i, value := range a {
  fmt.Println(i, value)  // 注意value是遍历的元素副本
}
for i, _ := range a {
  fmt.Println(a[i])
}
```



### `if` 判断

```go
if init; cond {
   // init 声明的变量只在if作用于中有效
}
```

`if-else`

```go
if v := math.Pow(x, n); v < lim{
  return v
}else{
  fmt.Printf("%g >= %g\n", v, lim) // init声明的变量在else中也有效
}
```



### `switch`分支

```go
switch init; key {
case "a":
  // logic a
case "b":
  // logic b
default:
  // default logic
}
```

特点：

- 只会运行选定的case，相当于默认有break
- switch的case无需为常量，取值也不限于整数
- 从上到下求值，匹配成功后停止
- 可以省略init和key, 只使用case的判断条件，用来代替复杂的`if-then-else`



### `defer`语句

`defer`语句会将函数推迟到外层函数返回后执行

推迟调用的函数会立即求值，但直到外层函数返回前该函数都不会被调用

```go
func main(){
  defer fmt.Println("world")
  fmt.Println("hello")
}
```

`defer`调用的函数会被压入一个栈中，外层函数返回时，按FILO顺序调用。



更多信息查看官方[blog](https://go.dev/blog/defer-panic-and-recover)

## 结构体、切片和映射

### 指针

从c++和go的指针对比开始学习比较好

1. **语法和声明**：
   - 在C++中，指针使用`*`符号定义，例如：
     ```cpp
     int* p;
     ```
   - 在Go中，指针使用`*`符号指示指针类型，**但没有`*`用于变量声明，而是用`&`取地址**，例如：
     ```go
     var p *int
     ```

2. **指针算术**：
   
   - C++支持指针算术，可以直接对指针进行加减运算，改变指针的地址。例如：
     ```cpp
     p++; // 移动到下一个整数的位置
     ```
   - **Go不支持指针算术**。这是为了提高安全性，避免许多常见的指针错误。
   
3. **空指针**：
   - 在C++中，空指针可以用`nullptr`（C++11及以后的版本）或`NULL`表示。
   - 在Go中，空指针直接用`nil`表示。

4. **内存管理**：
   - C++要求开发者手动管理内存，包括动态分配和释放。使用`new`和`delete`来管理内存。
   - Go有**自动垃圾回收机制**（GC），大部分情况下不需要手动管理内存。

5. **递归引用**：
   - 在C++中，结构体或类可以包含指向自己类型的指针，形成递归引用。
   - 在Go中，结构体也可以包含指向自己类型的指针，但由于语言设计，通常使用结构体和切片等更安全的方式来实现复杂的数据结构。

6. **指针传递**：
   
   - 在C++中，函数参数可以通过指针传递，允许通过指针修改调用者的变量值。
   - 在Go中，函数参数都是通过值传递（值的副本），如果想要修改传入的值，需要传递指针。
   
7. **接口的实现**：
   - 在C++中，指针和引用可以实现多态，但在Go中，接口的实现可以被值接收和指针接收，因此 Go 允许灵活的值和指针使用。

总的来说，Go的指针设计更加注重安全性和简洁性，避免了C++中指针相关的一些常见错误和复杂性。在使用时，开发者需要根据不同的语言特点来合理处理指针和引用。

```go
i, j = 42, 2701

p := &i // 指向i
fmt.Println(*p) // 通过指针读取i的值
*p = 21 // 通过指针设置i的值
fmt.Println(i) // 

p = &j // p改成指向j
*p = *p / 37  // 通过指针进行运算
fmt.Println(j)
```

### 结构体

**定义**

```go
type Vertex struct {
	X int
	Y int
}
```

结构体就是一组字段。

结构体字段可以通过点`.`访问

结构体字段也可以通过指针`.`访问，go支持使用隐式解引用。

```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
  p.X = 1e9 // (*p).X 也能用
	fmt.Println(v)
}
```



**结构体自变量**

结构体的初始化有多种方式

这里说的是 `[Name] : [value]`这种方式声明初始化的变量，没有声明的使用默认值。

```go
type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体
	v2 = Vertex{X: 1}  // Y:0 被隐式地赋予零值
	v3 = Vertex{}      // X:0 Y:0
	p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）
)
```

另外，go结构体中的变量不支持设置默认值，只能使用“构造函数”实现（普通函数）。



### 数组

定义

`var a [2]string`

定义+初始化

`primes := [6]int{2, 3, 5, 7, 11}`

索引

`a[0] = "Hello"`



### 切片

切片为数组元素提供更灵活的视角，实际中切片比数组更常用。

`[] T`表示一个元素类型为`T`的切片

切片通过两个下标来界定，和python一样，`a[low : high]`（半闭半开）



**切片就像数组的引用，切片不存储任何数据，只是描述了底层数组中的一段。更改切片会修改底层的数组，同样其他共享底层数组的切片也会被修改！！！**

```go
package main

import "fmt"

func main() {
	a := [4]int{1, 2, 3, 4}
	fmt.Println(a)

	slice1 := a[0:2]
	slice2 := a[1:3]
	fmt.Println(slice1, slice2)

	slice1[1] = 1
	fmt.Println(slice2)
}

```



**切片字面量**

A slice literal is like an array literal without the length

```go
[3]bool{true, true, false} // array literal
[]bool{true, true, false} // slice literal
```

使用字面量定义，底层也会创建一个数组，可以用反射获取，细节以后再说。留一个[blog](https://blog.frognew.com/2021/11/read-go-sources-slice.html)



**切片默认行为**

下面行为等价

```go
a[0:10]
a[:10]
a[0:]
a[:]
```



**切片长度和容量**

长度就是它所包含的元素个数。通过`len(s)`获取。

切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。通过`cap(s)`获取。

切片可以重新切片扩展长度。



**nil切片**

切片的零值是`nil`



**`make`创建切片**

创建时，传入参数依次是类型，长度和容量

`a := make([]int, 5)`

`a := make([]int, 0, 5)`



**切片的切片**

切片可以包含类型，当然也包含其他切片。

```go
board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
}
```

string类型本身是`[]byte`类型的切片，但没有`cap`属性，不能通过下标进行修改，但是可以通过整体赋值进行修改，底层是创建了一个新`[]byte`数组。



**向切片追加元素**

`func append(s []T, vs ... T) []T`

如果底层数组大小太小，不足以容纳元素，会分配一个更大的数组，返回的切片指向这个新分配的数组。



练习

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
  pic := make([][]uint8, dx)
  for i := 0; i < dx; i++ {
    pic[i] = make([]uint8, dy)
    for j := 0; j < dy; j++ {
      pic[i][j] = uint8((i + j )/ 2)
    }
  }
  return pic
}

func main() {
  pic.Show(Pic)
}
```

### 映射

映射就是map/dict

映射的零值是`nil`, 即没有键，也不能添加键。

`make` 函数会返回给定类型的映射，并将其初始化备用。

```go
type Vertex struct {
  Lat, long float64
}

var m map[string]Vertex

func main(){
  m = make(map[string]Vertex)
  m["Bell Labs"] = Vertex{40.68433, -74.39967}
  fmt.Println(m["Bell Labs"])
}
```



映射字面量，和结构体类似，只不过必须有键名.

```go
var m = map[string]Vertex{
  "Bell Labs": Vertex{40.68433, -74.39967},
  "Google" : Vertex{37.42202, -122.08408})
}
```

如果顶层类型是一个类型名，在字面量中可以省略。

映射用法

```go
m[key] = elem // 修改
elem = m[key] // 获取
delete(m. key) // 删除元素
elem, ok := m[key] // 双赋值检测元素是否存在
```

map练习

```go
// word count
func WordCount(s string) map[string]int{
  sp := strings.Split(s, " ")
  res := make(map[string]int])
  for _, substr := range sp{
    res[substr] += 1
  }
  return res
}
```



### 函数值

函数也是一种值，可以像其他值一样传递

函数值可以用作函数的参数或返回值



```go
func compute(fn func(float64, float64) float64) float64 {
  return fn(3,4)
}
```

**函数闭包**

闭包也是一种函数值，该函数被绑定到一些变量上，可以访问并赋值引用的变量。

```go
func adder() func(int) int {
  sum := 0
  return func(x int) int {
    sum += x
    return sum
  }
}
```

**fibonacci**闭包

```go
func fibonacci() func() int {
  a, b := 0, 1
  return func() int {
    a, b = b, a+b
    return a
  }
}

func main (){
  f := fibonacci()
  for i := 0; i < 10; i++{
    fmt.Println(f())
  }
}
```



## 方法和接口

go没有类，通过struct和接口进行OOP设计

### 方法

类方法实现通过带**接收者**参数的函数实现。

接受者在它自己的参数列表内，位于func关键字和方法名之间。

```go
type Vertex struct {
  X, Y float64
}

func (v Vectex) Abs() float64{
  return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main(){
  v := Vertex{3, 4}
  fmt.Println(v.Abs())
}
```

**方法就是一个带接收者的函数**。

也可以为非接收者声明方法，在接收者列表传入类型即可，但接收者的类型定义和方法声明必须在同一包里。





**指针类型的接收者**

指针接收者的方法可以修改接收者指向的值（如这里的 `Scale` 所示）。 由于方法经常需要修改它的接收者，指针接收者比值接收者更常用。

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```

如果把`Scale`函数的星号去掉，最后的返回值是防缩前的结果。

**函数指针参数和非指针参数**

```go
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func ScaleFunc(v Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

```

指针接收者的方法可以修改接收者指向的值（如这里的 `Scale` 所示）。 由于方法经常需要修改它的接收者，指针接收者比值接收者更常用。

**方法与指针重定向**

接受一个值作为参数的函数必须接受一个指定类型的值：

```
var v Vertex
ScaleFunc(v, 5)  // 编译错误！
ScaleFunc(&v, 5) // OK
```

而接收者为指针的的方法被调用时，接收者既能是值又能是指针：

```
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```

这种情况下，方法调用 `p.Abs()` 会被解释为 `(*p).Abs()`。



为什么用指针作为**接收者**？

1. 可以修改接收者的值
2. 避免调用时的复制，更高效

最好不要混用指针和非指针



### 接口

Go 接口（interface）是 Go 语言中的一个重要特性，用于定义一组方法的集合。接口不需要具体的实现，而是用于描述某个类型应当具有哪些方法。这种设计使得 Go 语言可以支持多态和动态类型的编程。

```go
type Speaker interface { // 接口
    Speak() string
}

type Dog struct{}
func (d Dog) Speak() string {
    return "Woof!"
}

type Cat struct{}
func (c Cat) Speak() string {
    return "Meow!"
}

func MakeItSpeak(s Speaker) {
    fmt.Println(s.Speak())
}

```



**隐式实现**

接口的实现不需要使用`implements`等关键字显示声明，只要实现一个接口的所有方法，就算实现了这个接口。接口的定义可以出现在任何包中。

```go
type I interface {
	M()
}

type T struct {
	S string
}

// 此方法表示类型 T 实现了接口 I，不过我们并不需要显式声明这一点。
func (t T) M() {
	fmt.Println(t.S)
}
```

**接口值**

接口值可以用作函数的参数或返回值。

在内部，接口值可以看做包含值和具体类型的元组：

```
(value, type)
```

```go
package main

import (
	"fmt"
	"math"
)

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	fmt.Println(t.S)
}

type F float64

func (f F) M() {
	fmt.Println(f)
}

func main() {
	var i I

	i = &T{"Hello"} // *T实现了接口I，可以赋值
	describe(i)
	i.M()

	i = F(math.Pi) // F也实现了接口
	describe(i) 
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

接口值保存了一个具体底层类型的具体值。接口值调用方法时会执行其底层类型的同名方法。

**底层值为`nil`的接口值**

即便接口内的具体值为`nil`, 方法仍会被`nil`接收者调用（同类型）。

*这里还有一个`nil != nil`问题，可以去搜索了解*

在一些语言中，这会触发一个空指针异常，但在 Go 中通常会写一些方法来优雅地处理它（如本例中的 `M` 方法）。

**注意:** 保存了 nil 具体值的接口其自身并不为 nil。

```go
func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```



接口和接口值有什么区别？

- **接口**定义了一组方法，是一个类型的抽象和契约
- **接口值**是一个具体的值与其动态类型的组合



**空接口**

指令了0个方法的接口称为空接口

```go
interface {}
```

空接口可保存任何类型的值。（因为每个类型都至少实现了零个方法。）

空接口被用来处理未知类型的值。例如，`fmt.Print` 可接受类型为 `interface{}` 的任意数量的参数。



**类型断言**

类型断言提供了访问接口值底层具体值的方式。

`t := i.(T)`

该语句断言接口值 `i` 保存了具体类型 `T`，并将其底层类型为 `T` 的值赋予变量 `t`。若 `i` 并未保存 `T` 类型的值，该语句就会触发一个 panic。

如何避免panic？

为了 **判断** 一个接口值是否保存了一个特定的类型，类型断言可返回两个值：其底层值以及一个报告断言是否成功的布尔值。

```
t, ok := i.(T)
```

若 `i` 保存了一个 `T`，那么 `t` 将会是其底层值，而 `ok` 为 `true`。

否则，`ok` 将为 `false` 而 `t` 将为 `T` 类型的零值，程序并不会产生 panic。

请注意这种语法和读取一个映射时的相同之处。

```go
package main

import "fmt"

func main() {
	var i interface{} = "hello"

	s := i.(string)
	fmt.Println(s)

	s, ok := i.(string)
	fmt.Println(s, ok)

	f, ok := i.(float64)
	fmt.Println(f, ok)

	f = i.(float64) // panic
	fmt.Println(f)
}

```

**类型选择**

**类型选择** 是一种按顺序从几个类型断言中选择分支的结构。

它们针对给定接口值所存储的值的类型进行比较。

```
switch v := i.(type) {
case T:
    // v 的类型为 T
case S:
    // v 的类型为 S
default:
    // 没有匹配，v 与 i 的类型相同
}
```

类型选择与一般的 switch 语句相似，不过类型选择中的 case 为类型（而非值），类型选择中的声明与类型断言 `i.(T)` 的语法相同，只是具体类型 `T` 被替换成了关键字 `type`。

此选择语句判断接口值 `i` 保存的值类型是 `T` 还是 `S`。在 `T` 或 `S` 的情况下，变量 `v`会分别按 `T` 或 `S` 类型保存 `i` 拥有的值。在默认（即没有匹配）的情况下，变量 `v`与 `i` 的接口类型和值相同。

```go
package main

import "fmt"

func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("二倍的 %v 是 %v\n", v, v*2)
	case string:
		fmt.Printf("%q 长度为 %v 字节\n", v, len(v))
	default:
		fmt.Printf("我不知道类型 %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
```

**常用接口**

Stringer

Readers

rot13Reader

Image



## 泛型

### 类型参数

可以使用类型参数编写 Go 函数来处理多种类型。 函数的类型参数出现在函数参数之前的方括号之间。

```
func Index[T comparable](s []T, x T) int
```

此声明意味着 `s` 是满足内置约束 `comparable` 的任何类型 `T` 的切片。 `x` 也是相同类型的值。

`comparable` 是一个有用的约束，它能让我们对任意满足该类型的值使用 `==` 和 `!=` 运算符。在此示例中，我们使用它将值与所有切片元素进行比较，直到找到匹配项。 该 `Index` 函数适用于任何支持比较的类型。

### 泛型类型

除了泛型函数之外，Go 还支持泛型类型。 类型可以使用类型参数进行参数化，这对于实现通用数据结构非常有用。

此示例展示了能够保存任意类型值的单链表的简单类型声明。

```go
// List 表示一个可以保存任何类型的值的单链表。
type List[T any] struct {
	next *List[T]
	val  T
}
```

## 并发

### 协程

协程（goroutine）是由 Go 运行时管理的轻量级线程。

`go f(x, y, z)`

上面代码启动一个新的协程并执行

`f, x, y, z`的求值发生在当前协程，`f`的执行发生在新的协程

go协程在相同的地址空间中进行，因此访问共享元素必须考虑同步，`sync`包提供了同步的机制，但我们有更好用的`channel`。



### 通道

信道是带有类型的管道，你可以通过它用信道操作符 `<-` 来发送或者接收值。

```go
ch <- v    // 将 v 发送至信道 ch。
v := <-ch  // 从 ch 接收值并赋予 v。
```

**（“箭头”就是数据流的方向。）**

和映射与切片一样，信道在使用前必须创建：

```
ch := make(chan int)
```

默认情况下，发送和接收操作在另一端准备好之前都会阻塞。这使得 Go 程可以在没有显式的锁或竞态变量的情况下进行同步。

以下示例对切片中的数进行求和，将任务分配给两个 Go 程。一旦两个 Go 程完成了它们的计算，它就能算出最终的结果。

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 发送 sum 到 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // 从 c 接收

	fmt.Println(x, y, x+y)
}
```

**带缓冲的通道**

信道可以是 **带缓冲的**。将缓冲长度作为第二个参数提供给 `make` 来初始化一个带缓冲的信道：

```
ch := make(chan int, 100)
```

仅当通道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接受方会阻塞。

下面的情况会出现死锁

```go
func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	ch <- 3
	fmt.Println(<-ch)
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```



**range 和 close**

发送者可以关闭一个通道表示没有需要发送的值，接收者可以通过接收表达式第二个返回值，检测通道是否关闭

```go
v, ok := <-ch // 通道关闭，ok为false
```



循环`for-range`会一直从信道中接收值，直到关闭

```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

```

**信道应该由发送者关闭而不是接收者，向一个已关闭的信道发送值会引发panic。**

**信道与文件不同，通常不需要关闭，只有在必须通知接收者不再有数据需要发送时才会关闭。**



**select**

`select` 语句使一个 Go 程可以等待多个通信操作。

`select` 会阻塞到某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会随机选择一个执行。



select 是 Go 中的一个控制结构，类似于 switch 语句。

select 语句只能用于通道操作，每个 case 必须是一个通道操作，要么是发送要么是接收。

select 语句会监听所有指定的通道上的操作，一旦其中一个通道准备好就会执行相应的代码块。

如果多个通道都准备好，那么 select 语句会随机选择一个通道执行。如果所有通道都没有准备好，那么执行 default 块中的代码

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

