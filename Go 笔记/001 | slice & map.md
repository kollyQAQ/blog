[TOC]

## 切片

### 内部实现

切片是基于数组实现的，它的底层是数组，它自己本身非常小，可以理解为对底层数组的抽象。因为基于数组实现，所以它的底层的内存是连续分配的，效率非常高，还可以通过索引获得数据，可以迭代以及垃圾回收优化的好处。

切片对象非常小，是因为它是只有 3 个字段的数据结构：一个是指向底层数组的指针，一个是切片的长度，一个是切片的容量。这 3 个字段，就是 Go 语言操作底层数组的元数据，有了它们，我们就可以任意的操作切片了。

```go
func main() {

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:3]

	newSlice[0] = 10

	fmt.Println(slice)
	fmt.Println(newSlice)

	fmt.Println("========================================================")

	/*
		对于底层数组容量是k的切片slice[i:j]来说
		长度：j-i
		容量:k-i
		此外还有一种 3 个索引的方法，第 3 个用来限定新切片的容量，其用法为 slice[i:j:k]
	*/
	slice1 := []int{1, 2, 3, 4, 5}
	newSlice1 := slice1[1:3]
	newSlice2 := slice[1:2:3]

	fmt.Printf("newSlice1长度:%d,容量:%d\n", len(newSlice1), cap(newSlice1))
	fmt.Printf("newSlice2长度:%d,容量:%d\n", len(newSlice2), cap(newSlice2))

	fmt.Println("========================================================")

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
	fmt.Printf("len=%d cap=%d type=%T &s=%p %v\n", len(s), cap(s), s, &s, s)
}
```

```bash
[1 10 3 4 5]
[10 3]
========================================================
newSlice1长度:2,容量:4
newSlice2长度:1,容量:2
========================================================
len=6 cap=6 type=[]int &s=0xc0000a8000 [2 3 5 7 11 13]
len=0 cap=6 type=[]int &s=0xc0000a0040 []
len=4 cap=6 type=[]int &s=0xc0000a00a0 [2 3 5 7]
len=2 cap=4 type=[]int &s=0xc0000a0100 [5 7]
```



## Map

### 内部实现

Map 是给予散列表来实现，就是我们常说的 Hash 表，所以我们每次迭代 Map 的时候，打印的 Key 和 Value 是无序的，每次迭代的都不一样，即使我们按照一定的顺序存在也不行。

Map 的散列表包含一组桶，每次存储和查找键值对的时候，都要先选择一个桶。如何选择桶呢？就是把指定的键传给散列函数，就可以索引到相应的桶了，进而找到对应的键值。



这种方式的好处在于，存储的数据越多，索引分布越均匀，所以我们访问键值对的速度也就越快，当然存储的细节还有很多，大家可以参考 Hash 相关的知识，这里我们只要记住 **Map 存储的是无序的键值对集合**。

#### 修改

在映射 `m` 中插入或修改元素：

```
m[key] = elem
```

### 读取

```
elem = m[key]
```

### 删除

```
delete(m, key)
```

### 通过双赋值检测某个键是否存在

在 Go Map 中，如果我们**获取一个不存在的键的值，也是可以的，返回的是值类型的零值**，这样就会导致我们不知道是真的存在一个为零值的键值对呢，还是说这个键值对就不存在。对此，Map 为我们提供了检测一个键值对是否存在的方法

```
elem, ok := m[key]
```

若 `key` 在 `m` 中，`ok` 为 `true` ；否则，`ok` 为 `false`。

### 遍历 & 排序

想要遍历 Map 的话，可以使用 `for range` 风格的循环，和遍历切片一样。

```go
dict := map[string]int{"张三": 43}
for key, value := range dict {
	fmt.Println(key, value)
}
```

这里的 `range` 返回两个值，第一个是 Map 的键，第二个是 Map 的键对应的值。这里再次强调，这种遍历是无序的，也就是键值对不会按既定的数据出现，如果想安顺序遍历，可以先对 Map 中的键排序，然后遍历排序好的键，把对应的值取出来，下面看个例子就明白了。

```go
func main() {
	dict := map[string]int{"王五": 60, "张三": 43}
	var names []string
	for name := range dict {
		names = append(names, name)
	}
	sort.Strings(names) //排序
	for _, key := range names {
		fmt.Println(key, dict[key])
	}
}
```

这个例子里有个技巧，`range` 一个 Map 的时候，也可以使用一个返回值，这个默认的返回值就是 Map 的键

