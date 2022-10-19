# 【Go 语言圣经】方法


{{< admonition quote >}}
本文内容来自：[Go语言圣经（中文版）](https://golang-china.github.io/gopl-zh/)
{{< /admonition >}}

**环境配置：**
- 系统：*Windows11*
- 编辑器：*vscode*

{{< admonition  >}}
本文假设你已经安装了Go并配置好相关环境，如果你还没有安装Go，请前往Go官方网站进行[下载安装](https://golang.google.cn/dl/)
{{< /admonition >}}

#### 💐 方法声明
*在函数声明时，在其名字之前放上一个变量，即是一个方法。*

***

第一个Distance的调用实际上用的是包级别的函数geometry.Distance，而第二个则是使用刚刚声明的Point，调用的是Point类下声明的Point.Distance方法。

这种p.Distance的表达式叫做选择器，因为他会选择合适的对应p这个对象的Distance方法来执行。
```go
package geometry

import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q))  // "5", method call
```

#### 🌸 基于指针对象的方法
当调用一个函数时，会对其每一个参数值进行拷贝。避免进行这种默认的拷贝，这种情况下我们就需要用到指针
```go
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```
想要调用指针类型方法(*Point).ScaleBy，只要提供一个Point类型的指针即可，像下面这样。
```go
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // "{2, 4}"
```
如果接收器p是一个Point类型的变量，并且其方法需要一个Point指针作为接收器，我们可以用下面这种简短的写法：
```go
p.ScaleBy(2)
```

#### 💮 封装
Go语言只有一种控制可见性的手段：大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。这种限制包内成员的方式同样适用于struct或者一个类型的方法。因而如果我们想要封装一个对象，我们必须将其定义为一个struct。

提前预留一部分空间，来避免反复的内存分配。
```go
type Buffer struct {
    buf     []byte
    initial [64]byte
    /* ... */
}

// Grow expands the buffer's capacity, if necessary,
// to guarantee space for another n bytes. [...]
func (b *Buffer) Grow(n int) {
    if b.buf == nil {
        b.buf = b.initial[:0] // use preallocated space initially
    }
    if len(b.buf)+n > cap(b.buf) {
        buf := make([]byte, b.Len(), 2*cap(b.buf) + n)
        copy(buf, b.buf)
        b.buf = buf
    }
}
```

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E6%96%B9%E6%B3%95/  

