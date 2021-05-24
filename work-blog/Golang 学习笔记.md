## Golang 学习笔记

### 1. 基本语法

#### 1.1 声明变量

**标准格式：**

```go
var 变量名 类型 = 表达式
```

示例代码：

```go
var hp int = 100  // 
var array []float32  =  make([]float, 32)// 定义数组
```

**短变量声明**

var 的变量还有一种更简单的写法：

```go
hp := 100
```

但是需要注意，这种推导声明的方式**要求左边的变量没有定义过的，不然会编译报错。**当做多个返回值时，要保证至少有一个变量是没有被定义过的。

**匿名变量**

在使用多重赋值时，如果不需要在左值中接收变量，可以使用匿名变量 `_` 进行替代。

```go
a,_ := GetData()
```

匿名变量不会占用命名空间，也不会分配内存。

#### 1.2 数据类型

`Go` 语言中除了基本的整型、浮点型、布尔型、字符串外，还有切片、结构体、函数、`map`、通道(`channel`)。

- 整型分为：`int8`、`int16`、`int32`、`int64`，其中 `int16` 类似于 `short` 类型，`int64` 类似于 `long` 类型
- 浮点型分为：`float32` 和 `float64`
- 布尔型：`boolean`
- 切片：就是数组，声明格式：`var name []T`

**类型转换**

在 `GO` 中，使用类型前置加括号进行类型转换。一般格式如下：

`T(表达式)`

示例代码：

```go
func main() {
	var c float32 = math.Pi
	fmt.Println(int(c))
}
```

**指针**

每个变量在运行时都有一个地址，这个地址代表变量在内存中的位置。`Go` 语言使用 `&` 操作符放在变量前面对变阿玲进行取地址操作。

```go
ptr := &v   // v 的类型为 T
```

其中 `v` 代表被去地址的变量，被取地址的 `v` 使用 `ptr` 变量进行接收，`ptr` 的类型就是 `*T`，称作 `T` 的指针类型，`*T` 代表指针。取地址操作符 `&` 和取值操作符 `*` 是一对互补操作符，`&` 去除地址，`*` 根据地址取出指向的值。

**interface{} 代表任意类型的数据**

#### 1.3 集合

##### **1. 数组**

数组的定义：

```go
var 数组变量名 [元素数量]T
```

数组的初始化参照：

```go
var team = [3]string{"hammer","soldier","mum"}
var team = [...]string{"hammer","soldier","mum"} //让编译器确定数组大小

// 数组的遍历
for i := 0; i < len(team); i++ {
	fmt.Println(team[i])
}

for key, value := range team {
	fmt.Println(key, value)
}
```

##### **2. 切片**

切片是指默认指向一段连续内存的区域，可能是数组，也可能是切片本身。格式如下：

```go
slice [开始位置 : 结束位置]

声明一个切片
var name[]T
```

同样也可以使用 `make()` 函数构造切片，格式如下：

```go
make([]T, size, cap)
```

- T：切片的元素类型
- size：切片的大小，分配多少个元素
- cap：预分配的元素个数，该值不影响 size，只是提前分配空间，降低多次分配时的性能问题

##### **3. map**

在 `Go` 语言中，`map` 的定义是这样的：

```go
map[KeyType]ValueType
```

- `KeyType`：为键类型
- `ValueType`：值类型

```go
scene := make(map[string]int)  // map 在使用时需要使用 make 进行主动创建
scene["route"] = 66
```

示例代码：

```go
func main() {

	var testsMap = make(map[string]string)
	testsMap["dengshiwei"]="testAB"
	testsMap["loginid"] = "loginAB"
	fmt.Println(testsMap["loginid"])
	// 特殊的写法，通过 ok 判断读取的 key 是否存在
	v,ok := testsMap["distinct"]
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("key is not exist.")
	}
}
```

在上面的使用过程中，我们可以通过这种特殊的写法判断键值是否存在。

**遍历**

在对于 `map` 的遍历，同时使用 `range` 函数。

```go
for key,value := range testsMap {
	fmt.Println("key = " + key + ", value = " + value)
}
```

**删除元素**

使用 `delete()` 内建函数从 map 中删除键值对。格式如下：

```go
delete(map, 键)
```

如果需要清空一个 `map`，最好的办法就是重新 `make` 创建一个。

**并发条件下使用 map**

在并发条件下使用  `map` 时，需要通过 `sync.Map` 进行创建。`Go` 语言中的 `map` 在并发条件下，只读是线程安全的，同时读写是线程不安全的。`sync.Map` 具有以下特性：

- 无需初始化，直接声明；
- Sync.Map 不能使用 map 的方式进行取值和设置操作，而是使用 `sync.Map` 提供的 `Store` 和 `Load`、`Delete` 进行操作；
- 使用 `Range` 函数配合一个回调函数进行遍历，回调中返回 `true`，表示继续迭代，`false` 标识终止迭代；

```go
func main() {
	var scene sync.Map
	scene.Store("greece", "color")
	scene.Store("london", "colorLondon")
	scene.Store("encrypt", "test")

	v, ok := scene.Load("encrypt")
	if ok {
		fmt.Println(v)
	}

	scene.Range(func(key, value interface{}) bool {
		fmt.Println(key, value)
		return true
	})
}
```

**4. 列表 list**

列表是一种非连续的存储容器。列表的初始化格式：

```go
变量名 := list.New()  // 方式 1
var 变量名 list.List // 方式 2
```

#### 1.3 流程控制

##### **1. 条件判断（if）**

格式：

```go
if 表达式1 {
  
} else if 表达式2 {
  
}
```

**特殊写法**

在 `if` 表达式之前添加一个执行语句，根据变量的值进行判断。示例：

```go
if err := Connect(); err != nil {
  fmt.Println(err)
  return
}
```

在上面的示例中，`err != nil` 才是 `if` 的判断表达式。

##### 2. 构建循环（for）

for 循环的格式如下：

```go
for 初始化语句;条件表达式;结束语句 {
  循环体代码
}
```

- 初始化语句，一般用于变量的初始化，当然也可以忽略不写
- 条件表达式，控制是否循环的开关。条件表达式如果被忽略，则默认无限循环

示例：

```go
func main() {

	for step := 2; step < 10; step++ {

	}

	step := 0
	for ; step < 10; step++ {

	}

	for ; ; step++ {
		if step > 10 {
			
		}
	}
	
	// 无限循环
	for {
		fmt.Println("无限循环")
	}
	
	// 只有一个循环条件的循环
	for step>10 {
		
	}
}
```

##### 3. for range 键值循环

用于数组、切片的遍历。

```go
func main() {
	c := make(chan int)
	go func() {
		c <- -1
		c <- 2
		c <- 3
		close(c)
	}()

	for v := range c {
		fmt.Println(v)
	}
}
```

### 2. 函数

#### 2.1 函数声明格式

```go
func 函数名(参数名) (返回值列表) {
  函数体
}
```

在 `Go` 语言中支持多返回值，多返回值能够方便活动函数执行后的参数返回。通常使用最后一个返回参数作为函数执行过程中出现的问题异常。

在 `Go` 语言中，同样可以将一个函数赋值给一个变量：

```go
func fire(a int) string{
	return "number is"
}

func main() {
	var f func(int) string
	f = fire
	f(1)
}
```

**定义匿名函数**

`Go` 函数指没有函数名的函数，可以在使用的时候进行声明。声明格式如下：

```go
func(参数列表) (返回值列表) {
  函数体
}
```

同样匿名函数可以在调用时进行使用，也可以赋值给变量，或者作为回调函数。

```go
func main() {
	// 在定义时调用匿名函数
	func(data int) {
		fmt.Println("hello", data)
	}(100)

	// 将匿名函数赋值给变量
	f := func(int) {
		fmt.Println("hello")
	}
	f(2)

	// 函数作为参数
	visit(199, f)

}

// 函数作为参数
func visit(data int, f func(int)) {
	f(1)
}
```

**函数类型实现接口**

总体来看有点面向对象的味道。

```go
func main() {
	var invoker Invoker
	s := new (Struct)
	invoker = s
	invoker.Call("hello")
}

type Invoker interface {
	Call(interface{})
}

type Struct struct {

}

func (s *Struct) Call(p interface{}) {
	fmt.Println("from struct", p)
}
```

在上面的示例中，我们声明了一个接口 `Invoker`，然后定义了一个接口的实现 `Struct`，在接口的实现中，实现了 `Call` 接口，所以在最终 `main` 方法中，我们可以讲一个 `Struct` 类型变量赋值给 `Invoker` 接口变量。

#### 2.2 闭包

闭包对它作用域上部的变量进行修改，修改引用的变量就会对变量的实际值进行修改。

```go
func main() {
	generator := playerGen("Jams")
	name,mood := generator()
	fmt.Println(name, mood)

}

// 定义一个返回函数的方法，然后内部使用闭包实现
func playerGen(name string) func() (string, int) {
	hp := 150
	return func() (string, int) {
		return name, hp
	}
}
```

#### 2.3 defer 延迟语句

`defer` 语句会进行延迟到函数结束时调用，当出现多个 `defer` 语句时，则是按照倒序执行。**通常可用于资源释放、释放文件句柄。**

```go
var (
	valueByKey = make(map[string]interface{})
	// 保证安全的互斥锁
	valueLock sync.Mutex
)

func main() {
	valueLock.Lock()
	defer valueLock.Unlock()
	v := valueByKey["test"]
	fmt.Println(v)
}
```

#### 2.4 错误异常处理

在`Go` 语言中，异常错误的处理思想包含以下特征：

- 一个可能造成错误的函数，需要在返回值中返回一个错误接口(`error`)，如果调用成功，错误接口返回 `nil`
- 函数调用后需要检查错误，如果发生错误，必须对错误进行处理

### 3. 结构体

结构体是类型中带有成员的复合类型。结构体的成员是由一系列的成员变量组成，这些成员变量也成为字段。结构体的定义格式如下：

```go
type 类型名 struct {
  字段1 类型
  字段2 类型
}
```

示例：

```go
type Point struct {
	x int
	y int
}
```

#### 3.1 结构体实例化

结构体本身是一种类型，所以可以通过 `var` 关键字进行声明。

```go
var ins T  //声明一个实例化
ins := new(T)  //创建指针类型的结构体
ins := &T{}   // 取结构体的地址代表依次 new 实例化
```

示例代码：

```go
func main() {
	var player1 Player
	player1.HealthPoint = 123
	player1.magicPoint = 29
	player1.Name = "Player1"

	player2 := new(Player)
	player2.Name = "Player2"

	player3 := &Player{}
	player3.Name = "Player3"
}

type Player struct {
	Name string
	HealthPoint int
	magicPoint int
}
```

#### 3.2 初始化结构体成员变量

```go
	player4 := &Player{
		Name: "Player4",
		HealthPoint: 5,
		magicPoint: 6,
	}

//初始化匿名结构体
	ins := struct {
		id int
		data string
	}{1024, "hello world"}
```

#### 3.3 带有继承关系的构造

```go
type Player struct {
	Name string
	HealthPoint int
	magicPoint int
}

type Basketball struct {
	Player
}
```

#### 3.4 结构体添加方法

```go
func main() {
	var player1 Player
	player1.HealthPoint = 123
	player1.magicPoint = 29
	player1.Name = "Player1"
	// 调用结构体的方法
	player1.getPlayerInfo()
}

type Player struct {
	Name string
	HealthPoint int
	magicPoint int
}

func (player Player) getPlayerInfo()(string,int,int) {
	return player.Name, player.HealthPoint, player.magicPoint
}
```

### 4. 接口

接口也成协议，是定义的一套方法接口。接口声明的格式：

```go
type 接口类型名 interface {
  方法名1(参数) 返回值1
}
```

在定义接口时，一般会添加 `er` 结尾。比如 `Writer`、`Closer`

#### 4.1 实现接口的方法

- 接口的方法与实现接口的类型方法格式一致
- 接口中所有方法都需要实现

```go

func main() {
	file := new(FileWriter)
	file.WriteData("hello")
}

type DataWriter interface {
	WriteData(data interface{}) error
	CanWriter() bool
}

type FileWriter struct {

}

func (fileWriter FileWriter) WriteData(data interface{}) error {
	fmt.Println("写入数据:" , data)
	return nil
}
```

#### 4.2 接口类型断言

`Go` 语言中使用接口类型断言用于接口的转换。类型断言的基本格式如下：

```go
t := i.(T)
```

- i：代表接口变量
- T：代表转换的目标类型
- t：代表转换后的变量

示例：

```go
t,ok := i.(T)
```

### 5. 并发

在 `Go` 语言中，可以使用关键字 `go` 创建协程。格式如下：

```go
go 函数名(参数列表)
```

- 函数名：要调用的函数
- 参数列表：需要传入的参数列表

这样通过 `go` 关键字就可以将一个函数放到协程中进行执行。

**匿名函数创建 goroutine**

```go
go func(参数列表) {
  函数体
}(调用的参数列表)
```

#### 5.1 多个 goroutine 中的通信

在多个 `goroutine` 中通过通道（channel）进行通信。声明格式：

```go
var 通道变量 chan 通道类型
```

- 通道变量：保存通道的变量
- 通道类型：通道内的数据类型

**创建通道**

通道是引用类型，需要使用 `make` 关键字进行创建。格式如下：

```go
通道实例 := make(chan 数据类型)
```

示例：

```go
ch1 := make(chan int)
```

**通道发送数据**

```go
通道变量 <- 值
```

**通道接收数据**

```go
<- 通道变量

// 阻塞接收数据
data := <- ch
// 非阻塞接收数据
data, ok := <- ch
```

注意通道的数据接收时，如果没有接收方处理，则会持续阻塞。因此通道的接收必定会在另一个 goroutine 中进行。如果接收方在接收时，通道中没有发送方发送数据，接收方也会发生阻塞，知道发送方发送数据位置。

示例：

```go
func main() {
	ch := make(chan int)
	go func() {
		for i := 3; i >= 0; i-- {
			ch <- i
			time.Sleep(time.Second)
		}

	}()

	for data := range ch {
		fmt.Println(data)

		if data == 0 {
			break
		}
	}
}
```

#### 5.2 互斥锁（sync.Mutex）

通过互斥锁 `sync.Mutex` 保证同时只有一个 `goroutine` 可以访问共享资源。

```go
var(
	count int
	countLock sync.Mutex
)

func getCount() int {
	countLock.Lock()
	defer countLock.Unlock()
	return count
}

func writeCount(c int) {
	countLock.Lock()
	defer countLock.Unlock()
	count = c
}

func main() {
	writeCount(1)
	fmt.Println(getCount())
}
```

#### 5.3 读写互斥锁（sync.RWMutex）

在读多写少的环境汇总，可以优先使用读写互斥锁。

```go
var(
	count int
	countLock sync.RWMutex
)

func getCount() int {
	countLock.RLock()
	defer countLock.RUnlock()
	return count
}
```

#### 5.4 等待组（sync.WaitGroup）

保证在并发环境中完成指定数量的任务。

