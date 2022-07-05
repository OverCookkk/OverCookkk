---
title: map
tags: [map]      #添加的标签
categories: 
  - GO
description:。
toc: true
#cover: 
---



## map定义

Go语言中提供的映射关系容器为`map`，其内部使用`散列表（hash）`实现。map是一种**无序**的基于`key-value`的数据结构，Go语言中的map是**引用类型**，必须初始化才能使用。

map类型的变量默认初始值为nil，需要使用make()函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

其中cap表示map的容量，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。



## map基本使用

map中的数据都是成对出现的，map的基本使用示例代码如下：

```go
func main() {
	scoreMap := make(map[string]int, 8)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
}
```

map也支持在声明的时候填充元素，例如：

```go
func main() {
	userInfo := map[string]string{
		"username": "xxx",
		"password": "123456",
	}
}
```

Go语言中有个判断map中键是否存在的特殊写法，格式如下:

```go
value, ok := map[key]
```

Go语言中使用`for range`遍历map：

```go
for k, v := range scoreMap {
    fmt.Println(k, v)
}
```

**注意：** 遍历map时的元素顺序与添加键值对的顺序无关。



## 并发安全的sync.Map

如果要保证并发的安全，最朴素的想法就是使用锁，但是这意味着要把一些并发的操作强制串行化，性能自然就会下降。

### 核心思想

事实上，除了使用锁，还有一个办法，也可以达到类似并发安全的目的，就是原子操作（atomic）。sync.Map的设计非常巧妙，充分利用了atmoic的尽量使用原子操作和mutex的配合，最大程度上减少了锁的使用。

```go
type Map struct {
	mu Mutex
	read atomic.Value // readOnly
    dirty map[interface{}]*entry
	misses int
}

type readOnly struct {
	m       map[interface{}]*entry
	amended bool // true if the dirty map contains some key not in m.
}

// An entry is a slot in the map corresponding to a particular key.
type entry struct {
	p unsafe.Pointer // *interface{}
}
```

使用了两个原生的map作为存储介质，分别是read map和dirty map（只读字典和脏字典）。

- 只读字典使用atomic.Value来承载，保证原子性和高性能；脏字典则需要用互斥锁来保护，保证了互斥。

- 只读字典和脏字典中的键值对集合并不是实时同步的，它们在某些时间段内可能会有不同。

- 无论是read还是dirty，本质上都是map[interface{}]*entry类型，这里的entry其实就是Map的value的容器。

- entry的本质，是一层封装，可以表示具体值的指针，也可以表示key已删除的状态（即逻辑假删除）



### 架构设计图

![sysc.Map底层数据结构原理图](https://raw.githubusercontent.com/OverCookkk/PicBed/master/blogImg/sysc.Map%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%8E%9F%E7%90%86%E5%9B%BE.png)

- read map由于是原子包托管，主要负责高性能，但是无法保证拥有全量的key（因为对于新增key，会首先加到dirty中），所以read某种程度上，类似于一个key的快照。

- **dirty map拥有全量的key**，当Store操作要新增一个之前不存在的key的时候，预先是增加自dirty中的。

- 在查找指定的key的时候，总会先去只读字典中寻找，并不需要锁定互斥锁。只有当read中没有，但dirty中可能会有这个key的时候，才会在锁的保护下去访问dirty。

- 在存储键值对的时候，只要read中已存有这个key，并且该键值对未被标记为“expunged”，就会把新值存到里面并直接返回，这种情况下也不需要用到锁。

- expunged和nil，都表示标记删除，但是它们是有区别的，简单说expunged是read独有的，而nil则是read和dirty共有的。

- read和map的关系，是一直在动态变化的，可能存在重叠，也可能是某一方为空；重叠的公共部分，由分为两种情况，nil和normal。

- read和dirty之间是会互相转换的，在dirty中查找key对次数足够多的时候，sync.Map会把dirty直接作为read，即触发dirty=>read升级。同时在某些情况，也会出现read=>dirty的重塑。