# 【Go语言圣经】入门


{{< admonition quote >}}
本文内容来自：[Go语言圣经（中文版）](https://golang-china.github.io/gopl-zh/)
{{< /admonition >}}

**环境配置：**
- 系统：*Windows11*
- 编辑器：*vscode*

{{< admonition  >}}
本文假设你已经安装了Go并配置好相关环境，如果你还没有安装Go，请前往Go官方网站进行[下载安装](https://golang.google.cn/dl/)
{{< /admonition >}}

#### 🍇 Hello, World
**新建文件`helloWorld.go`**
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, 世界")
}
```
**执行命令：**
```shell
go run helloWorld.go
```
**控制台输出：**
```shell
Hello, 世界
```
**使用build子命令：**
```shell
go build helloWrold.go
```
命令执行完毕在当前目录生成`helloWorld.exe`文件

#### 🍉 命令行参数
**os包**

os 包以跨平台的方式，提供了一些与操作系统交互的函数和变量。程序的命令行参数可从 os 包的 Args 变量获取；os 包外部使用 os.Args 访问该变量。

**os.Args**

os.Args 变量是一个字符串（string）的 切片（slice）（译注：slice 和 Python 语言中的切片类似，是一个简版的动态数组）

os.Args 的第一个元素：os.Args[0]，是命令本身的名字；其它的元素则是程序启动时传给它的参数。

**slice**

s[m:n] 形式的切片表达式，产生从第 m 个元素到第 n-1 个元素的切片。

如果省略切片表达式的 m 或 n，会默认传入 0 或 len(s)，因此前面的切片可以简写成 os.Args[1:]。

**len函数**

用于计算数组(包括数组指针)、切片(slice)、map、channel、字符串等数据类型的长度

例子：
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    var s, sep string
    for i := 1; i < len(os.Args); i++ {
        s += sep + os.Args[i]
        sep = " "
    }
    fmt.Println(s)
}
```

**range关键字**

Go 语言中 range 关键字用于 for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素。在数组和切片中它返回元素的索引和索引对应的值，在集合中返回 key-value 对。

例子：
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    s, sep := "", ""
    for _, arg := range os.Args[1:] {
        s += sep + arg
        sep = " "
    }
    fmt.Println(s)
}
```

**空标识符（blank identifier）：`_`**

空标识符可用于在任何语法需要变量名但程序逻辑不需要的时候（如：在循环里）丢弃不需要的循环索引，并保留元素值。

**短变量声明：`:=`**

只能用在函数内部，而不能用于包变量。

例子：
```go
func main(){
  s := [...]int{1,2,3}
}
```

#### 🍅 查找重复的行
**map**

map 存储了键/值（key/value）的集合，对集合元素，提供常数时间的存、取或测试操作

**counts[input.Text()]++**

等价于：
```go
line := input.Text()
counts[line] = counts[line] + 1
```

**bufio 包**

使处理输入和输出方便又高效。

Scanner 类型是该包最有用的特性之一，它读取输入并将其拆成行或单词；通常是处理行形式的输入最简单的方法。

从程序的标准输入中读取内容：
```go
input := bufio.NewScanner(os.Stdin)
// 每次调用 input.Scan()，即读入下一行，并移除行末的换行符；
for input.Scan() {
    counts[input.Text()]++
}
```

**Printf 转换**

Printf 有一大堆这种转换，Go程序员称之为动词（verb）。
```
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

**查找重复的行：从标准输入中读取数据**
```go
// Dup1 prints the text of each line that appears more than
// once in the standard input, preceded by its count.
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    counts := make(map[string]int)
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}

```

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E5%85%A5%E9%97%A8/  

