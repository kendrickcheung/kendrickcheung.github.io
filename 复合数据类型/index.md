# 【Go 语言圣经】复合数据类型


{{< admonition quote >}}
本文内容来自：[Go语言圣经（中文版）](https://golang-china.github.io/gopl-zh/)
{{< /admonition >}}

**环境配置：**
- 系统：*Windows11*
- 编辑器：*vscode*

{{< admonition  >}}
本文假设你已经安装了Go并配置好相关环境，如果你还没有安装Go，请前往Go官方网站进行[下载安装](https://golang.google.cn/dl/)
{{< /admonition >}}

#### 🌶️ slice
Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已

一个slice由三个部分构成：指针、长度和容量。

#### 🥦 map
哈希表是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。

内置的make函数可以创建一个map：
```go
ages := make(map[string]int) // mapping from strings to ints
```
我们也可以用map字面值的语法创建map，同时还可以指定一些最初的key/value：
```go
ages := map[string]int{
    "alice":   31,
    "charlie": 34,
}
```
Map的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。在实践中，遍历的顺序是随机的，每一次遍历的顺序都不相同。
```go
import "sort"

var names []string
for name := range ages {
    names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
```
和slice一样，map之间也不能进行相等比较；唯一的例外是和nil进行比较。要判断两个map是否包含相同的key和value，我们必须通过一个循环实现：
```go
func equal(x, y map[string]int) bool {
    if len(x) != len(y) {
        return false
    }
    for k, xv := range x {
        if yv, ok := y[k]; !ok || yv != xv {
            return false
        }
    }
    return true
}
```

### 🍆 结构体
*结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。*
```go
type Employee struct {
    ID            int
    Name, Address string
    DoB           time.Time
    Position      string
    Salary        int
    ManagerID     int
}
```

**结构体字面值**

结构体值也可以用结构体字面值表示，结构体字面值可以指定每个成员的值。
```go
type Point struct{ X, Y int }

p := Point{1, 2}
```
如果考虑效率的话，较大的结构体通常会用指针的方式传入和返回
```go
func Bonus(e *Employee, percent int) int {
    return e.Salary * percent / 100
}
```

**结构体比较**

如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那样的话两个结构体将可以使用==或!=运算符进行比较。
```go
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{2, 1}
fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q)                   // "false"
```

**结构体嵌入和匿名成员**
```go
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}
```
访问每个成员显得繁琐：
```go
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20
```
Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。
```go
type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}
```
得益于匿名嵌入的特性，我们可以直接访问叶子属性而不需要给出完整的路径：
```go
var w Wheel
w.X = 8            // equivalent to w.Circle.Point.X = 8
w.Y = 8            // equivalent to w.Circle.Point.Y = 8
w.Radius = 5       // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```
结构体字面值必须遵循形状类型声明时的结构，所以我们只能用下面的两种语法，它们彼此是等价的：
```go
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
    Circle: Circle{
        Point:  Point{X: 8, Y: 8},
        Radius: 5,
    },
    Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:8, Y:8}, Radius:5}, Spokes:20}

w.X = 42

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:42, Y:8}, Radius:5}, Spokes:20}
```

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E5%A4%8D%E5%90%88%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/  

