1、需要注意`sync.WaitGroup`是一个结构体，进行参数传递的时候要传递指针。

2、Go语言中的`defer`语句会将其后面跟随的语句进行延迟处理。在`defer`归属的函数即将返回时，将延迟处理的语句按`defer`定义的**逆序**进行执行，也就是说，先被`defer`的语句最后被执行，最后被`defer`的语句，最先被执行。

在Go语言的函数中`return`语句在底层并不是原子操作，它分为给返回值赋值和RET指令两步。而`defer`语句执行的时机就在返回值赋值操作后，RET指令执行前。

defer经典案例

阅读下面的代码，写出最后的打印结果。

```go
func f1() int {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f2() (x int) {
	defer func() {
		x++
	}()
	return 5
}

func f3() (y int) {
	x := 5
	defer func() {
		x++
	}()
	return x
}
func f4() (x int) {
	defer func(x int) {
		x++	//改变的是函数中的x的副本
	}(x)
	return 5
}
func main() {
	fmt.Println(f1())
	fmt.Println(f2())
	fmt.Println(f3())
	fmt.Println(f4())
}
```

defer面试题

```go
func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	x := 1
	y := 2
    defer calc("AA", x, calc("A", x, y))	//先算calc("A", x, y)，得到值付给calc("AA",x,   )，执行defer压栈，此时压栈传入的x是1，不是下面的10
	x = 10
	defer calc("BB", x, calc("B", x, y))
	y = 20
}
```



3、

在做对外接口的数据对接和转换时，我们经常需要对 JSON 数据进行处理。

如下代码：

```go
type T struct {
 name string
 age  int
}

func main() {
 p := T{"煎鱼", 18}
 jsonData, _ := json.Marshal(p)
 fmt.Println(string(jsonData))
}
```

输出结果是空，原因是 JSON 的输出，只会输出公开（导出）字段，也就是结构体T里面的字段name和age首字母必须为大写。



4、

程序运行期间`funcB`中引发了`panic`导致程序崩溃，异常退出了。这个时候我们就可以通过`recover`将程序恢复回来，继续往后执行。

```go
func funcA() {
	fmt.Println("func A")
}

func funcB() {
	defer func() {
		err := recover()
		//如果程序出出现了panic错误,可以通过recover恢复过来
		if err != nil {
			fmt.Println("recover in B")
		}
	}()
	panic("panic in B")
}

func funcC() {
	fmt.Println("func C")
}
func main() {
	funcA()
	funcB()
	funcC()
}
```

**注意：**

1. `recover()`必须搭配`defer`使用。
2. `defer`一定要在可能引发`panic`的语句之前定义。



5、

在 Go 中，循环迭代器变量是一个单一的变量，在每个循环迭代中取不同的值。这如果使用不当，可能会导致非预期的行为。

如下代码：

```go
func main() {
 var out []*int
 for i := 0; i < 3; i++ {
  out = append(out, &i)
 }
 fmt.Println("Values:", *out[0], *out[1], *out[2])
 fmt.Println("Addresses:", out[0], out[1], out[2])
}
```

输出结果：

```text
Values: 3 3 3
Addresses: 0x40e020 0x40e020 0x40e020
```

原因是：在每次迭代中，我们将 i 的地址追加到 out 切片中，但由于它是同一个变量，我们实际上追加的是相同的地址，该地址最终包含分配给 i 的最后一个值。



6、for...range...拷贝副本

```go
func main() {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}

	for k, v := range m {
		fmt.Printf("k:%s v:%v", k, v)
	}
}
```

输出结果如下：

```text
k:zhou v:&{wang 22}
k:li v:&{wang 22}
k:wang v:&{wang 22}
```

原因：for range每次产生的k，v都是一个值拷贝，不是stus值对应的引用。

详解：for中，第一次循环，变量stu被赋值，指向"zhou"，然后放到map中保存，第二次循环，变量stu的被重新赋值，改为指向“li”，这时map中的指向也跟着改变。

解决方法：

```go
for _, stu := range stus{
    a := stu	//每次重新定义一个新的变量a
    m[stu.name] = &a
}
```

