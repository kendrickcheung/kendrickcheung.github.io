# 【Go 语言圣经】函数


{{< admonition quote >}}
本文内容来自：[Go语言圣经（中文版）](https://golang-china.github.io/gopl-zh/)
{{< /admonition >}}

**环境配置：**
- 系统：*Windows11*
- 编辑器：*vscode*

{{< admonition  >}}
本文假设你已经安装了Go并配置好相关环境，如果你还没有安装Go，请前往Go官方网站进行[下载安装](https://golang.google.cn/dl/)
{{< /admonition >}}

#### 🌶️ 函数声明
函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。
```go
func name(parameter-list) (result-list) {
    body
}

```
形式参数列表描述了函数的参数名以及参数类型。这些参数作为局部变量，其值由参数调用者提供。

#### 🥕 递归
**使用递归处理HTML文件**
1. 新建项目文件夹`chapter05`并用vscode打开
2. 在根目录`chapter05`下新建文件`fetch`
3. 在`fetch`下新建文件`main.go`:
```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path"
)

//!+
// Fetch downloads the URL and returns the
// name and length of the local file.
func fetch(url string) (filename string, n int64, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return "", 0, err
	}
	defer resp.Body.Close()

	local := path.Base(resp.Request.URL.Path)
	if local == "/" {
		local = "index.html"
	}
	f, err := os.Create(local)
	if err != nil {
		return "", 0, err
	}
	n, err = io.Copy(f, resp.Body)
	// Close file, but prefer error from Copy, if any.
	if closeErr := f.Close(); err == nil {
		err = closeErr
	}
	return local, n, err
}

//!-

func main() {
	for _, url := range os.Args[1:] {
		local, n, err := fetch(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch %s: %v\n", url, err)
			continue
		}
		fmt.Fprintf(os.Stderr, "%s => %s (%d bytes).\n", url, local, n)
	}
}
```
4. 在根目录`chapter05`下新建文件`findlinks1`
5. 在`findlinks1`下新建文件`main.go`:
```go
// Findlinks1 prints the links in an HTML document read from standard input.
package main

import (
    "fmt"
    "os"

    "golang.org/x/net/html"
)

func main() {
    doc, err := html.Parse(os.Stdin)
    if err != nil {
        fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
        os.Exit(1)
    }
    for _, link := range visit(nil, doc) {
        fmt.Println(link)
    }
}
// visit appends to links each link found in n and returns the result.
func visit(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && n.Data == "a" {
			for _, a := range n.Attr {
					if a.Key == "href" {
							links = append(links, a.Val)
					}
			}
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
			links = visit(links, c)
	}
	return links
}
```
5. 在根目录`chapter05`下执行命令：
```shell
go mod init chapter05
```
```shell
go mod tidy
```
```shell
go build chapter05/fetch
```
```shell
go build chapter05/findlinks1
```
```shell
./fetch https://baidu.com | ./findlinks1
```
6. 执行所有命令后在根目录`chapter05`下会生成文件`index.html`

#### 🍄 
在Go中，函数被看作第一类值（first-class values）：函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。
```go
    func square(n int) int { return n * n }
    func negative(n int) int { return -n }
    func product(m, n int) int { return m * n }

    f := square
    fmt.Println(f(3)) // "9"

    f = negative
    fmt.Println(f(3))     // "-3"
    fmt.Printf("%T\n", f) // "func(int) int"

    f = product // compile error: can't assign func(int, int) int to func(int) int
```

#### 🍚 匿名函数
函数字面量的语法和函数声明相似，区别在于func关键字后没有函数名。函数值字面量是一种表达式，它的值被称为匿名函数（anonymous function）。
```go
// squares返回一个匿名函数。
// 该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}
```

#### 🍠 可变参数
参数数量可变的函数称为可变参数函数。
```go
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
```

#### 🍜 Deferred函数
defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。
```go
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return ReadAll(f)
}
```
处理互斥锁（9.2章）:
```go
var mu sync.Mutex
var m = make(map[string]int)
func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}
```

#### 🍣 Panic异常
Go的类型系统会在编译时捕获很多错误，但有些错误只能在运行时检查，如数组访问越界、空指针引用等。这些运行时错误会引起panic异常。

不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。当某些不应该发生的场景发生时，我们就应该调用panic。
```go
switch s := suit(drawCard()); s {
case "Spades":                                // ...
case "Hearts":                                // ...
case "Diamonds":                              // ...
case "Clubs":                                 // ...
default:
    panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
}
```

#### 🍡 Recover捕获异常
如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。
```go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
```

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E5%87%BD%E6%95%B0/  

