本文主要说明我关于go语言数据结构的认识，站在一个广义的角度去理解这些数据类型，go的数据类型有：int, float, bool, map, slice, arr, struct, interface, chan, 它们都是来源于万物模具，go语言中一切类型都是interface类型，这个类似于python里面的type


### bool /int / uint / uintptr / float32 / complex64

###### 这些万物模型均为顶级类型，声明+初始化就是创造对象的过程

```go
var a bool = true
var b int = 8
var c uint64 = 1<<64 - 1
var d uintptr
var e float32 = 0.89
var f complex64 = cmplx.Sqrt(-5 + 12i)
```

### Array

array是万物模具，长度是数组类型的一部分( [4]int 和 [5]int 是完全不同的类型，元素类型也是数组类型的一部分 )

```go
// 定义
var a [5]int // 事实上数组的值会被初始化0
fmt.Println("array a:", a)

// 可以使用 array[index] = value 语法来设置数组指定位置的值，或者用 array[index] 得到值
a[1] = 10
a[3] = 30
fmt.Println("assign:", a)
fmt.Println("len:", len(a))

// 使用这个语法在一行内定义并初始化一个数组
b := [5]int{1, 2, 3, 4, 5}
fmt.Println("init:", b)
// 编译器也可以根据初始值的个数自行推断数组的长度
n := [...]int{1, 2}

// 遍历多维数组
var c [2][3]int
for i := 0; i < 2; i++ {
	for j := 0; j < 3; j++ {
		c[i][j] = i + j
	}
}
fmt.Println("2d: ", c)
```

### slice

slice是万物模具

存在的意义：数组的缺陷，长度一旦设定不得改变

###### 切片的特点：

1 切片是对数组的再次封装

> 关于这一点我们可以通过切片的一种定义方式看出来，底层还是依赖于数组
```go
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[1:4]
```
> 当然也可以通过make定义

2 每个数组的大小都是固定的。而切片则为数组元素提供动态大小的、灵活的视角

> 上面例子也可以得出，当然也可以通过append添加元素，实现动态变化长度

3 切片并不存储任何数据，它只是描述了底层数组中的一段

![切片只是底层数组的一段](https://s2.ax1x.com/2019/09/14/nyJEGt.png "切片只是底层数组的一段")

- 更改切片的元素会修改其底层数组中对应的元素

```go
names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
fmt.Println(names)

a := names[0:2]
b := names[1:3]
fmt.Println(a, b)

b[0] = "XXX"
fmt.Println(a, b)
fmt.Println(names)

// 打印结果
[John Paul George Ringo]
[John Paul] [Paul George]
[John XXX] [XXX George]
[John XXX George Ringo]
```

4 切片的长度与容量
> 切片的长度就是它所包含的元素个数。
> 切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。
> 切片 s 的长度和容量可通过表达式 len(s) 和 cap(s) 来获取。
> 通过重新切片来扩展一个切片，给它提供足够的容量

```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// 截取切片使其长度为 0
	s = s[:0]
	printSlice(s)

	// 拓展其长度
	s = s[:4]
	printSlice(s)
	// 舍弃前两个值
	s = s[2:]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

```

5 用make创建切片

```go
// 切片可以用内建函数 make 来创建，这也是你创建动态数组的方式。
// make 函数会分配一个元素为零值的数组并返回一个引用了它的切片：

a := make([]int, 5)  // len(a)=5
// 要指定它的容量，需向 make 传入第三个参数：

b := make([]int, 0, 5) // len(b)=0, cap(b)=5
b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]
```

6 切片可以包含任意类型，甚至是切片类型

```go
// 案例1
type A struct {}

func main() {
	a1 := A{}
	a2 := A{}
	a3 := A{}
	board := [][]A{
		[]A{a1, a1, a1},
		[]A{a2, a2, a2},
		[]A{a3, a3, a3},
	}
	fmt.Println(board)
}

// 案例2
func main() {
	// 创建一个井字板（经典游戏）
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// 两个玩家轮流打上 X 和 O
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
}
```

7 向切片追加元素

```go
func main() {
	var s []int
	printSlice(s)

	// 添加一个空切片
	s = append(s, 0)
	printSlice(s)

	// 这个切片会按需增长
	s = append(s, 1)
	printSlice(s)

	// 可以一次性添加多个元素
	s = append(s, 2, 3, 4)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

> 切片的本质，推荐阅读：[Go 切片：用法和本质](https://blog.go-zh.org/go-slices-usage-and-internals "Go 切片：用法和本质")

##### 关于new 与 make

```go
func new(Type) *Type
```

内建函数new分配内存。其第一个实参为类型，而非值。其返回值为指向该类型的新分配的零值的指针。

```go
func make(Type, size IntegerType) Type
```

内建函数make分配并初始化一个类型为切片、映射、或通道的对象。其第一个实参为类型，而非值。make的返回类型与其参数相同，而非指向它的指针。其具体结果取决于具体的类型：

1. 切片：size指定了其长度。该切片的容量等于其长度。切片支持第二个整数实参可用来指定不同的容量；它必须不小于其长度，因此 make([]int, 0, 10) 会分配一个长度为0，容量为10的切片。

1. 映射：初始分配的创建取决小的起始大小。

1. 通道：通道的缓存根据指定的缓存容量初始化。若 size为零或被省略，该信道即为无缓存的。


### map

map是一种万物模具

长相：

```go
map[KeyType]ValueType

// KeyType 可以是任意类型
// ValueType 可以是任意类型
```

创建map类型并实例化：

```go
type Vertex struct {
	Lat, Long float64
}
var m map[string]Vertex{}
var n map[int]int
// 事实上上面创建过程会自动实例化，值都是0值

// 也可以直接指定实例化
var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
var ppp = int(5)
```
修改映射：

```go
// 在映射 m 中插入或修改元素：
m[key] = elem

// 获取元素：
elem = m[key]

// 删除元素：
delete(m, key)

// 通过双赋值检测某个键是否存在：
elem, ok = m[key]
// 若 key 在 m 中，ok 为 true ；否则，ok 为 false。
// 若 key 不在映射中，那么 elem 是该映射元素类型的零值。

```

### func

func是一种万物模具，人们叫它“一等公民”

长相：

```go
func 函数名(参数)(返回值){
    函数体
}
```

创建func类型并实例化：

```go
type funType func(int, int) int

func add(x, y int) int {
	return x + y
}
funcObj := add
// funcObj就是funType类型的对象

// 事实上我们一般定义一个函数，并不会预先自定义一个函数类型，因为go语言内部会默认根据我们的函数“签名”定义一个对应的类型
```

匿名函数：

```go
// 将匿名函数保存到变量
add := func(x, y int) {
	fmt.Println(x + y)
}
add(10, 20) // 通过变量调用匿名函数

//自执行函数：匿名函数定义完加()直接执行
func(x, y int) {
	fmt.Println(x + y)
}(10, 20)
```

闭包

```go
// Go 函数可以是一个闭包。闭包是一个函数值，它引用了其函数体之外的变量。该函数可以访问并赋予其引用的变量的值，换句话说，该函数被这些变量“绑定”在一起。

// 例如，函数 adder 返回一个闭包。每个闭包都被绑定在其各自的 sum 变量上。
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

### struct

struct 是一种万物模具，这是个有情怀的家伙

长相：

```go
type Class struct Person {}
```

创建struct 并实例化：

```go
// 定义Vertex类型
type Vertex struct {
	X, Y float64
}

// 方法
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// 实例化
v = Vertex{2.1, 3.8}
```

方法与指针重定向1：
```go
// 定义一个类型Vertex
type Vertex struct {
	X, Y float64
}

// 方法为指针类型接受者
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

// 函数参数为指针类型
func ScaleFunc(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	// 实例化一个值类型对象
	v := Vertex{3, 4}
	// v是值类型，仍然可以调用Scale，因为v.Scale(2) 转化为 (&v).Scale(2)
	v.Scale(2)
	// 参数必须为指针类型
	ScaleFunc(&v, 10)
	// 实例化指针类型对象
	p := &Vertex{4, 3}
	// p本身就是指针类型
	p.Scale(3)
	// 参数必须为指针类型
	ScaleFunc(p, 8)
	fmt.Println(v, p)
}
```

方法与指针重定向2：
```go
// 定以一个类型Vertex
type Vertex struct {
	X, Y float64
}

// 方法为值类型接受者
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// 函数参数为值类型
func AbsFunc(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	// 实例化一个值类型对象
	v := Vertex{3, 4}
	// v是值类型，可以直接调Abs
	fmt.Println(v.Abs())
	// 参数必须为值类型
	fmt.Println(AbsFunc(v))
	// 实例化一个指针类型对象
	p := &Vertex{4, 3}
	// p是指针类型，仍然可以调值类型方法，因为p.Abs() 会被解释为 (*p).Abs()
	fmt.Println(p.Abs())
	// 参数必须为值类型
	fmt.Println(AbsFunc(*p))
}
```

### interface(非空interface)

> 非空interface 是万物模型，与struct联合实现多态，当然interface可以实现继承

#### 作用

> 实现多态

#### interface 经典案例

```go
//---------- 接 口 --------//
type shape interface {
    area() float64 //计算面积
    perimeter() float64 //计算周长
}
//--------- 长方形 ----------//
type rect struct {
    width, height float64
}
func (r *rect) area() float64 { //面积
    return r.width * r.height
}
func (r *rect) perimeter() float64 { //周长
    return 2*(r.width + r.height)
}
//----------- 圆  形 ----------//
type circle struct {
    radius float64
}
func (c *circle) area() float64 { //面积
    return math.Pi * c.radius * c.radius
}
func (c *circle) perimeter() float64 { //周长
    return 2 * math.Pi * c.radius
}
// ----------- 接口的使用 -----------//
func interface_test() {
    r := rect {width:2.9, height:4.8}
    c := circle {radius:4.3}
    s := []shape{&r, &c} //通过指针实现
    for _, sh := range s {
        fmt.Println(sh)
        fmt.Println(sh.area())
        fmt.Println(sh.perimeter())
    }
}
```
