---
title: "Go入门笔记"
date: 2016-12-15 22:00
updated: 2017-01-01 21:00
log: "增加教程"
---

[TOC]

## 教程

* [A Tour of Go](https://tour.golang.org/)
* [Go by Example](https://gobyexample.com/)
* [Go入门指南](http://wiki.jikexueyuan.com/project/the-way-to-go/) / [Github](https://github.com/Unknwon/the-way-to-go_ZH_CN)
* [Go 语言程序设计](https://book.douban.com/subject/24869910/)
* [怎么学习golang？](https://www.zhihu.com/question/23486344)里面给出了一些不错的入门资料


## 笔记

2017-01-01: 目前是官网入门指南[A Tour of Go](https://tour.golang.org/list)的笔记, 大部分代码来至上面.

### GOROOT 和 GOPATH

关于`GOROOT`和`GOPATH`环境变量, 如果是系统默认安装, 而非自定义的安装目录, 则`GOROOT`不需要设置.

> GOROOT must be set only when installing to a custom location. [from](https://golang.org/doc/install#install)

关于`GOPATH`, 必须设置, 在get/build/install包时用到, 第三方的包都会装在这个目录下, 包括里面的二进制文件, 所以建议将`$GOPATH/bin`加入到`$PATH`环境变量中. 更多可以`go help gopath`.


### package和import

一个基本的例子:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World")
}
```

其中package我的理解是继续或者说基于的包名; main包表示这个程序的执行入口, 编译时会将这个编译为可执行程序(`go build`)而不是$GOPATH/pkg/下的静态库(`go install`). 类似Python中的`if __name__ == '__main__'`.

import则表示导入要使用的标准库包或第三方包.

参考:

* [Understanding Golang Packages](http://thenewstack.io/understanding-golang-packages/)
* [golang-book Packages](https://www.golang-book.com/books/intro/11)

多个import语句可以使用打包导入(factored import), 更优雅:

```go
import (
	"fmt"
	"math"
)

// 等价
import "fmt"
import "match"
```

### Exported names

在 Go 中，首字母大写的名称是被导出的. 当import包时, 不被导出的包是无法被访问使用的.

### Function

```go
func add(x, y int) int {
	return x + y
}

func swap(x, y string) (string, string) {
	return y, x
}
```

函数名后面圆括号接参数, 参数后面指定返回类型, 多个相同的类型**不能**省略只写一个; 单个返回值可以不用圆括号括起来; 没有返回值则不写.

Named return values(命令返回值), 即在函数名后的返回值指定变量名, 函数体内只需要裸return即可:

```go
func swap(x, y string) (m, n string) {
    m, n = y, x
    return
}
```

这里不能对参数做修改饭后通过命名返回值返回, 否则报错:

```go
// 错误的
func swap(x, y string) (x, y string) {
    x, y = y, x
    return
}
```

函数参数, 闭包没啥好写的


### Variable, Constants and Type

几种定义方式:

```go
var i, j bool
var x, y int = 1, 2

func main() {
    var m, n = "abc", 100
    z := 200
    fmt.Println(i, j, x, y, m, n, z)
}
// 输出: false false 1 2 abc 100 200
```

Go中声明(declaration)和定义(definition)个人理解是不做区分的(和C不一样), 因为如果声明了变量但是未显式赋值, 会隐式复制给各类型的初始值(zero value, 零值).

另外`var var_name var_type`和函数参数一样, 变量名在前, 类型在后.

另外也可以不指定类型名, Go会根据赋值判断相应类型

最后, 也可以不写`var`, 改用`:=`的简明赋值语句, 但是此语法**只能用于函数内**, 而`var`则可以在函数外使用.

声明和导入包一样, 可以打包声明:

```go
var (
    i bool    = false
    j int     = 1
    z float64 = 0.3
)
```

关于类型关键词, 其中`byte`是`uint8`的别名, `rune`是`int32`的别名.

一般情况下, 数字用`int`即可.

一些基本类型的零值:

* 数字: 0
* 布尔: false
* 字符串: ""

Go中类型的转换用`T(v)`, 将值v转换为类型T:

```go
i := 1
f := float64(i)
```

Go中类型转换必须显示指定(C中是可以做隐式转换的).

常量类型变量声明, 不过使用关键字`const`, 不能使用`:=`语法

```go
const World = '世界'

const (
	Big   = 1 << 100
	Small = Big >> 99
)
```

### for / if / switch

`for`支持几种语法:

```go
// init statement; condition expression; post statement
// 另外这里注意只有后自增, 没有前自增; 写C/C++时习惯了用前自增, 这里总写错
for i := 1; i <= 10; i++ {
	fmt.Println(i)
}

// init / post statement 可以省略
sum := 1
for ; sum <= 10; {
	sum += sum
}

// 上面的例子, 前后两个分号`;`也可以省略, 这就是while的语法了
sum := 1
for sum <= 10 {
	sum += sum
}

// 上面的例子, 退出条件也省略, 就是无线循环了
for {
}
```

`if`语法:

```go
// condition expression
if i <= 10 {
	...
}

// if也支持init statement, if初始化的变量作用域只在if主体内
if i := 5; i <= 10 {
	...
}

// if ... else if ... else
if i := 5; i <= 3 {
	fmt.Println("<= 3")
} else if i <= 6 {
	fmt.Println("<= 6")
} else {
	fmt.Println("> 6")
}
```

`switch`语法:

```go
// 同样支持初始化语法
// 和C不同, 每个case语句的行为是自动`break`, 不需要手动写`break`
// 如果想保持和C的行为一致, 即匹配后还继续往下执行, 则可以在case中加上`fallthrough`
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		fmt.Printf("%s.", os)
}

// 如果switch condition没写, 则默认表示 `true`
// 行为和if ... else if ... else 一样
t := time.Now()
switch {
case t.Hour() < 12:
	fmt.Println("Good morning!")
case t.Hour() < 17:
	fmt.Println("Good afternoon.")
default:
	fmt.Println("Good evening.")
}
```

注:

* Go中初始、条件等语句不需要用`()`阔起来
* 主体部分必须用花括号`{}`阔起来


### defer

`defer`语句用于延迟函数的执行直到当前函数return, 但是defer的参数会立刻生成.

多个defer语句会进行压栈, 最后执行时是LIFO:

```go
fmt.Println("begin")

for i := 0; i < 3; i++ {
	defer fmt.Println(i)
}

fmt.Println("done")
// 返回: begin -> end -> 2 -> 1 -> 0
```


### Pointers

```go
i := 100
var p1 *int  // 如果没有初始化, 则零值是`nil`
p1 = &i
p2 := &i
*p2 = 101
fmt.Println(i, *p1, *p2)
```

* `*T`在声明时表示指向值T的指针
* `&`用于对值`取址`
* `*p*`在使用时表示对指针的`解引用`, 即取指针指向的值.

这里`*`需要注意, 在不同地方的含义不一样.


### Struct

结构体用法:

```go
type Vertex struct {
	X int
	Y int
}

var v Vertex = Vertex{3, 5} // 结构体初始化
v.X = 6                     // 通过dot获取结构体字段
fmt.Println(v)
p := &v    // 结构体指针
(*p).X = 7 // 结构体指针获取结构体字段
fmt.Println(v)
p.X = 8 // 上面的用法太笨拙, 这个更简单
fmt.Println(v)
// 输出:
// {6 5}
// {7 5}
// {8 5}

// Struct Literals 不清楚这个怎么翻译比较合适?
v1 := Vertex{X: 1} // X: 1, Y: 0
v2 := Vertex{}     // X: 0, Y: 0
p := &Vertex{1, 2} // has type *Vertex
fmt.Println(v1, v2, p)
// 输出: {1 0} {0 0} &{1 2}
// 注意p的输出结构式带有`&`, 表示输出的是结构体指针

```

### Array

数组是定长的`[n]T`

```go
var a1 [2]string
a1[0] = "hello"
a1[1] = "world"
a2 := [3]int{1, 2, 3}
fmt.Println(a1, a2)
// 输出: [hello world] [1 2 3]
```

注意长长度`[n]`也是类型的一部分


### Slice

数组是定长的, Go还提供了切片这个数据结构, 长度是动态变化的, 所以这个用的比数组更频繁.

(刚看到 slice/切片 这个词, 第一反应是一个函数, 结果是一个数据结构...)

因为长度是动态变化, 所以声明是`[]T`, 括号中不写.

Go中做切片(这里是动词)操作, 返回的是切片.

```go
array := [3]int{1, 2, 3}
var slice1 []int = array[1:3]
slice2 := array[0:2]
fmt.Println(slice1, slice2)
// import "reflect"
fmt.Println(reflect.TypeOf(slice1), reflect.TypeOf(slice2))
// 输出:
// [2 3] [1]
// []int []int
```

切片自身并不存储数据, 它是对底层数组的引用.

所以对切片中数据的修改, 会影响相应的底层数组的值, 也会影响其它引用到这个数组的切片

```
// 接上面的例子
slice1[0] = 100
fmt.Println(array, slice1, slice2)
// 输出: [1 100 3] [100 3] [1 100]
```

切片字面值(slice literal)和数组字面值(array literal)一样, 只是不需要指定长度:

```go
slice1 := []int{1, 2, 3}
slice2 := []struct {
	i int
	b bool
}{
	{1, true},
	{2, false},
	{3, true},  // 注意最后的逗号不能省略
}

fmt.Println(slice1, slice2)
// 输出: [1 2 3] [{1 true} {2 false} {3 true}]
```

切片的使用和Python类似, 支持:

```go
s[0:10]
s[:10]
s[0:]
s[:]
```

切片有length和capacity的概念

* length通过`len(s)`获取, 表示切片中元素的个数
* capacity通过`cap(s)`获取, 表示切片引用的**底层数组**中元素的个数, 从切片的第一个元素开始计算

下面这个例子比较有意思, 感觉容易入坑:

```go
func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

func main() {
    s := []int{1, 2, 3, 4, 5}
    printSlice(s)

    s = s[:0]
    printSlice(s)

    s = s[:4]  // 注意这里还可以扩展, 因为是引用, 底层数组一直存在
    printSlice(s)

    s = s[2:5] // 这里capacity就减少了
    printSlice(s)
}
// 输出
// len=5 cap=5 [1 2 3 4 5]
// len=0 cap=5 []
// len=4 cap=5 [1 2 3 4]
// len=3 cap=3 [3 4 5]
```

切片的零值是`nil`:

```go
var s []int  // 注意和s := []int{}不一样, 这个是空切片, 赋值过的
fmt.Println(s, len(s), cap(s))
if s == nil {
	fmt.Println("slice is nil")
}
```

`make`函数可以用来创建切片, 并指定length(必选)和capacity(可选):

```go
s1 := make([]int, 3)
printSlice(s1)

s2 := make([]int, 3, 5)
printSlice(s2)
// 输出
// len=3 cap=3 [0 0 0]
// len=3 cap=5 [0 0 0]
```

二维切片(类似C中二维数组):

```go
// len(s) = 2, cap(s) = 2
s := [][]string{
	[]string{"a", "b"},
	[]string{"c", "d"},
}
```

Go对切片的append操作提供了内置函数`append(s []T, v1, v2, v3, ...T) []T`, 最后返回append后的切片; 因为切片大小是动态的, 所以如果capacity不够, 会自动扩容:

```go
s := make([]int, 1, 3)
printSlice(s)

s = append(s, 1)
printSlice(s)

s = append(s, 2, 3)
printSlice(s)
// 输出:
// len=1 cap=3 [0]
// len=2 cap=3 [0 1]
// len=4 cap=6 [0 1 2 3]
```

for循环切片:

```go
s := []int{1, 2, 3}
// i 是切片索引, v是值的一个copy
for i, v := range s {
	fmt.Println(i, v)
}
```

和Python一样, 如果不关心索引, 可以直接赋值给变量名`_`


### Maps

映射(也就是字典吧)表示一个key/value对集合, 声明语法:

```go
var <map_name> map[<map_key_type>]<map_value_type>
```

即:

```go
type Vertex struct {
    X int
    Y int
}

var m map[string]Vertex  // 声明

func main() {
    m = make(map[string]Vertex)  // 创建
    m["a"] = Vertex{3, 4}
    m["b"] = Vertex{1, 2}
    fmt.Println(m)
}
// 输出: map[a:{3 4} b:{1 2}]
```

只声明的map, 零值是nil, nil map不能添加key/values, 即没有下面的make则后面不能操作.

声明、创建这块也可以直接写为`m := map[string]Vertex`

映射字面值(map literal):

```go
var m = map[string]Vertex{
	"a": Vertex{3, 4},
	"b": Vertex{1, 2},  // 注意逗号
}

// TODO: If the top-level type is just a type name, you can omit it from the elements of the literal.
// 什么是顶级类型?
// 下面是简写, 省去内部的值类型
var m = map[string]Vertex{
	"a": {3, 4},
	"b": {1, 2},
}
```

map的几个操作:

```go
// 修改某个key的value
m[k] = v

// 获取某个key的value
v = m[k]

// 删除某个key
delete(m, k)

// 测试某个key是否在m中
// ok是boolean, 存在则为true, 否则是false
v, ok = m[k]
```


### Methods

Go没有类, 但是可以给自定义类型(如结构体)定义方法(methods).

method和function类似, 只不过多了一个特殊的接收者参数(receiver), 位置在func关键字和method name之间.

```go
// 如这里定义Abs这个方法, 属于Vertex这个结构体, `(v Vertex)`就是function没有的多出的部分
func (v Vertex) AbsMethod() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
// 调用
v := Vertex{3, 4}
fmt.Println(v.AbsMethod())

// 这个是function, 将Vertex结构体当普通参数
func AbsFunc(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
// 调用
v := Vertex{3, 4}
fmt.Println(AbsFunc(v))
```

如上所说, 不光结构体可以声明方法, 比如:

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
```

但是receiver的类型必须定义在当前包里, 不能给其它包里定义的类型声明method, 比如内置类型.

上面使用的receiver是一个值(value receiver), receiver还是是一个指针(pointer receiver), 如:

```go
// method
func (v *Vertex) ScaleMethod(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

// function
func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

因为Go里函数参数是传值, 相当于一份拷贝, 所以如果使用`func (v Vertex) Scale...`而不是`func (v *Vertex) Scale...`, 则实际v.X和v.Y的值并没有被改变.

关于这块调用Scale时针对pointer和value, 需要注意一个坑:

```go
var v Vertex

// 针对上面的Scale
ScaleFunc(v, 3)      // compile error!
ScaleFunc(v, 3)      // ok
v.ScaleMethod(3)     // ok, as (&v).ScaleMethod()
p := &v
p.ScaleMethod(3)     // ok

// 针对上上面的Abs
AbsFunc(v)           // ok
AbsFunc(&v)          // compile error!
v.AbsMethod()        // ok
p := &v
p.AbsMehod()         // ok, as (*p).AbsMethod()
```

为了方便, Go的解释器对method作了一些自动化处理, 如上例子, 不论是pointer receiver还是value receiver的方法, 都可以通过pointer或value来调用.

更倾向于选择使用pointer receiver的原因有两个:

1. method内部可以修改receiver的值
2. 不是按值传递, 所以更节省空间, 比如需要传递的是一个很大的结构体


### Interfaces

Go Tour上这块英文感觉有点绕, 需要多读几遍.

> An interface type is defined as a set of method signatures.

一个接口类型是一组method的定义的集合.

> A value of interface type can hold any value that implements those methods.

接口类型是一个抽象类型, 它的值可以是任何值, 只需要这个值实现了接口的methods

接口的好处就是将接口的定义和时间分离.

```go
// 接口类型
type I interface {
    M()
}

// 具体类型
type T struct {
    S string
}

func (t *T) M() {
    fmt.Println(t.S)
}

// 具体类型
type F float64

func (f F) M() {
    fmt.Println(f)
}

func main() {
    var i I

    i = &T{"Hello"}
    fmt.Printf("(%v, %T)\n", i, i)
    i.M()

    i = F(0.11)
    fmt.Printf("(%v, %T)\n", i, i)
    i.M()
}
// 输出
// (&{Hello}, *main.T)
// Hello
// (0.11, main.F)
// 0.11
```

一个接口需要挂载到一个底层具体类型上, 调用接口的方法实际就是调用底层具体类型的同名方法.

如果实际类型是nil (nil underlying value), 则接口也是nil

但如果接口类型是nil (nil interface value), 则无法调用它的method, 否则报错

空接口定义: `var i interface{}`, 接口i可以是任何值.

类型断言(type assertion)让接口值可以访问所挂载具体类型的值: `t := i.(T)`, 其中i是接口值, T是具体类型名：

```go
var i interface{} = "hello"

s := i.(string)
fmt.Println(s)

s, ok := i.(string)
fmt.Println(s, ok)

// 这个和之前的map test类似
f, ok := i.(float64)
fmt.Println(f, ok)

f = i.(float64) // panic
fmt.Println(f)
```

type switch 结合了接口类型和switch语句, 语法`i.(type)`, 和type assertion语法有点像.

```go
// i is interface
switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
}
```

fmt包中定义了Stringer接口, 方法String(), 如目前用到最多的fmt.Println()就根据类型的String()方法输出内容:

```go
// type Stringer interface {
//     String() string
// }
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	fmt.Println(a)
```
