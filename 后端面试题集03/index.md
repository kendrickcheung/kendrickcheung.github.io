# 【面试】后端面试题集03


{{< admonition  >}}
本文内容来源于：[【Github】后端面试题集](https://github.com/xiaobaiTech/golangFamily)
{{< /admonition >}}

#### 🧨 什么是channel，为什么它可以做到线程安全？
{{< details "展开查看详情" >}}
*Channel是Go中的⼀个核⼼类型，可以把它看成⼀个管道，通过它并发核⼼单元就可以发送或者接收数据进⾏通讯(communication),Channel也可以理解是⼀个先进先出的队列，通过管道进⾏通信。*

*Golang的Channel,发送⼀个数据到Channel 和 从Channel接收⼀个数据 都是 原⼦性的。⽽且Go的设计思想就是:不要通过共享内存来通信，⽽是通过通信来共享内存，前者就是传统的加锁，后者就是Channel。也就是说，设计Channel的主要⽬的就是在多任务间传递数据的，这当然是安全的。*
{{< /details >}}

#### ✨ 读写锁或者互斥锁读的时候能写吗?
{{<hide-text hide="答案：*Go中读写锁包括读锁和写锁，多个读线程可以同时访问共享数据；写线程必须等待所有读线程都释放锁以后，才能取得锁；同样的，读线程必须等待写线程释放锁后，才能取得锁，也就是说读写锁要确保的是如下互斥关系，可以同时读，但是读-写，写-写都是互斥的。*">}}

#### 🎈 Channel是同步的还是异步的.
{{< details "展开查看详情" >}}
*Channel是异步进⾏的。*

**channel存在3种状态：**
1. nil，未初始化的状态，只进⾏了声明，或者⼿动赋值为nil
2. nil，未初始化的状态，只进⾏了声明，或者⼿动赋值为nil
3. closed，已关闭，千万不要误认为关闭channel后，channel的值是nil
{{< /details >}}

#### 🎉 下列哪个类型可以使⽤ cap()函数？
- A. array
- B. slice
- C. map
- D. channel

{{<hide-text hide="答案：*A B D*；array 返回数组的元素个数；slice 返回 slice 的最⼤容量；channel 返回 channel 的容量；">}}

#### 🎊 Data Race问题怎么解决？能不能不加锁解决这个问题？
{{< details "展开查看详情" >}}
同步访问共享数据是处理数据竞争的⼀种有效的⽅法.golang在1.1之后引⼊了竞争检测机制，可以使⽤ go run -race 或者 go build -race来进⾏静态检测。 其在内部的实现是,开启多个协程执⾏同⼀个命令， 并且记录下每个变量的状态.

竞争检测器基于C/C++的ThreadSanitizer 运⾏时库，该库在Google内部代码基地和Chromium找到许多错误。这个技术在2012年九⽉集成到Go中，从那时开始，它已经在标准库中检测到42个竞争条件。现在，它已经是我们持续构建过程的⼀部分，当竞争条件出现时，它会继续捕捉到这些错误。

竞争检测器已经完全集成到Go⼯具链中，仅仅添加-race标志到命令⾏就使⽤了检测器。

```shell
$ go test -race mypkg // 测试包
$ go run -race mysrc.go // 编译和运⾏程序
$ go build -race mycmd // 构建程序
$ go install -race mypkg // 安装程序
```
*要想解决数据竞争的问题可以使⽤互斥锁sync.Mutex,解决数据竞争(Data race),也可以使⽤管道解决,使⽤管道的效率要⽐互斥锁⾼.*
{{< /details >}}

#### 🎋 如何在运⾏时检查变量类型？
{{<hide-text hide="答案：*类型开关是在运⾏时检查变量类型的最佳⽅式。类型开关按类型⽽不是值来评估变量。每个 Switch ⾄少包含⼀个case，⽤作条件语句，和⼀个 defaultcase，如果没有⼀个 case 为真，则执⾏。*">}}

#### 🎍 Go 两个接⼝之间可以存在什么关系？
{{<hide-text hide="答案：*如果两个接⼝有相同的⽅法列表，那么他们就是等价的，可以相互赋值。如果接⼝ A的⽅法列表是接⼝ B的⽅法列表的⾃⼰，那么接⼝ B可以赋值给接⼝ A。接⼝查询是否成功，要在运⾏期才能够确定。*">}}

#### 🎎 关于map，下⾯说法正确的是？
- A. map 反序列化时 json.unmarshal() 的⼊参必须为 map 的地址；
- B. 在函数调⽤中传递 map，则⼦函数中对 map 元素的增加不会导致⽗函数中 map 的修改；
- C. 在函数调⽤中传递 map，则⼦函数中对 map 元素的修改不会导致⽗函数中 map 的修改；
- D. 不能使⽤内置函数 delete() 删除 map 的元素

{{<hide-text hide="答案：A">}}

#### 🎏 关于同步锁，下⾯说法正确的是？
- A. 当⼀个 goroutine 获得了 Mutex 后，其他 goroutine 就只能乖乖的等待，除⾮该 goroutine 释放这个 Mutex；
- B. RWMutex 在读锁占⽤的情况下，会阻⽌写，但不阻⽌读；
- C. RWMutex 在写锁占⽤情况下，会阻⽌任何其他 goroutine（⽆论读和写）进来，整个锁相当于由该 goroutine独占；
- D. Lock() 操作需要保证有 Unlock() 或 RUnlock() 调⽤与之对应；

{{<hide-text hide="答案：*A B C*">}}

#### 🎐 Go 当中同步锁有什么特点？作⽤是什么
{{< details "展开查看详情" >}}
当⼀个 Goroutine（协程）获得了 Mutex 后，其他 Gorouline（协程）就只能乖乖的等待，除⾮该 gorouline 释放了该 MutexRWMutex在读锁占⽤的情况下，会阻⽌写，但不阻⽌读 RWMutex 在写锁占⽤情况下，会阻⽌任何其他

goroutine（⽆论读和写）进来，整个锁相当于由该 goroutine 独占同步锁的作⽤是保证资源在使⽤时的独有性，不会因为并发⽽导致数据错乱，保证系统的稳定性。
{{< /details >}}

#### 🎑 Go 语⾔当中 Channel（通道）有什么特点，需要注意什么？
{{<hide-text hide="答案：*如果给⼀个 nil 的 channel 发送数据，会造成永远阻塞如果从⼀个 nil 的 channel 中接收数据，也会造成永久爱阻塞给⼀个已经关闭的 channel 发送数据，会引起 pannic 从⼀个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回⼀个零值。*">}}

#### 🎀 Go 语⾔当中 Channel 缓冲有什么特点？
{{<hide-text hide="答案：*⽆缓冲的 channel 是同步的，⽽有缓冲的 channel 是⾮同步的。*">}}

#### 🎁 关于channel，下⾯语法正确的是?
- A. var ch chan int
- B. ch := make(chan int)
- C. <- ch
- D. ch <-

{{<hide-text hide="答案：*A B C*；解析：写 chan 时，<- 右端必须要有值">}}

#### 🎗️ Go 语⾔中 cap 函数可以作⽤于那些内容？
{{< details "展开查看详情" >}}
*cap 函数在讲引⽤的问题中已经提到，可以作⽤于的类型有：*
1. array(数组)
2. slice(切⽚)
3. channel(通道)
{{< /details >}}

#### 🎟️ go convey 是什么？⼀般⽤来做什么？
{{< details "展开查看详情" >}}
1. go convey 是⼀个⽀持 golang 的单元测试框架
2. go convey 是⼀个⽀持 golang 的单元测试框架
3. go convey 提供了丰富的断⾔简化测试⽤例的编写
{{< /details >}}

#### 🎖️ Go 语⾔当中 new 和 make 有什么区别吗？
{{< details "展开查看详情" >}}
*new的作⽤是初始化⼀个纸箱类型的指针 new函数是内建函数，函数定义：*
```go
func new(Type)*Type
```
1. 使⽤ new函数来分配空间
2. 传递给 new函数的是⼀个类型，⽽不是⼀个值
3. 返回值是指向这个新⾮配的地址的指针
{{< /details >}}

#### 🏆 Go 语⾔中 make 的作⽤是什么？
*make的作⽤是为 slice, map or chan 的初始化然后返回引⽤ make函数是内建函数，函数定义：*
```go
func make(Type, size IntegerType) Type
```
make(T, args)函数的⽬的和 new(T)不同仅仅⽤于创建 slice, map, channel ⽽且返回类型是实例。

#### 🏅 Printf(),Sprintf(),FprintF()都是格式化输出，有什么不同？
{{<hide-text hide="答案：*虽然这三个函数，都是格式化输出，但是输出的⽬标不⼀样 Printf 是标准输出，⼀般是屏幕，也可以重定向。Sprintf()是把格式化字符串输出到指定的字符串中。 Fprintf()是吧格式化字符串输出到⽂件中。*">}}

#### ⚽ Go 语⾔当中数组和切⽚的区别是什么？
{{< details "展开查看详情" >}}
数组：数组固定⻓度数组⻓度是数组类型的⼀部分，所以[3]int 和[4]int 是两种不同的数组类型数组需要指定⼤⼩，不指定也会根据处初始化对的⾃动推算出⼤⼩，不可改变数组是通过值传递的

切⽚：切⽚可以改变⻓度切⽚是轻量级的数据结构，三个属性，指针，⻓度，容量不需要指定⼤⼩切⽚是地址传递（引⽤传递）可以通过数组来初始化，也可以通过内置函数 make()来初始化，初始化的时候 len=cap，然后进⾏扩容。
{{< /details >}}

#### ⚾ Go 语⾔当中值传递和地址传递（引⽤传递）如何运⽤？有什么区别？举例说明
{{< details "展开查看详情" >}}
1. 值传递只会把参数的值复制⼀份放进对应的函数，两个变量的地址不同，不可相互修改。
2. 地址传递(引⽤传递)会将变量本身传⼊对应的函数，在函数中可以对该变量进⾏值内容的修改。
{{< /details >}}

#### 🥎 Go 语⾔当中数组和切⽚在传递的时候的区别是什么？
{{<hide-text hide="答案：*1. 数组是值传递；2.  切⽚是引⽤传递*">}}

#### 🏀 Go 语⾔是如何实现切⽚扩容的？
{{< details "展开查看详情" >}}
```go
func main(){
 arr := make([]int,0)
 for i := 0; i < 2000; i++{
 fmt.Println("len 为", len(arr),"cap 为", cap(arr)) arr = append(arr, i)
 }
}
```
结果：*依次是0,1,2,4,8,16,32,64,128,256,512,1024 但到了1024 之后,就变成了1024,1280,1696,2304 每次都是扩容了四分之⼀左右*
{{< /details >}}

#### 🏈 关于 channel 下⾯描述正确的是？
- A. 向已关闭的通道发送数据会引发 panic；
- B. 从已关闭的缓冲通道接收数据，返回已缓冲数据或者零值；
- C. ⽆论接收还是接收，nil 通道都会阻塞；
- D. close() 可以⽤于只接收通道；
- E. 单向通道可以转换为双向通道；
- F. 不能在单向通道上做逆向操作（例如：只发送通道⽤于接收）；

{{<hide-text hide="答案：*A B C F*">}}

#### 🏉 看下⾯代码的 defer 的执⾏顺序是什么？ defer 的作⽤和特点是什么？
{{< details "展开查看详情" >}}
**defer 的作⽤是：**

你只需要在调⽤普通函数或⽅法前加上关键字 defer，就完成了 defer 所需要的语法。当 defer 语句被执⾏时，跟在 defer 后⾯的函数会被延迟执⾏。直到包含该 defer 语句的函数执⾏完毕时，defer 后的函数才会被执⾏，不论包含 defer 语句的函数是通过 return 正常结束，还是由于 panic 导致的异常结束。你可以在⼀个函数中执⾏多条defer 语句，它们的执⾏顺序与声明顺序相反。

**defer 的常⽤场景：**
1. defer 语句经常被⽤于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。
2. 通过 defer 机制，不论函数逻辑多复杂，都能保证在任何执⾏路径下，资源被释放。
3. 释放资源的 defer 应该直接跟在请求资源的语句后。
{{< /details >}}

#### 🎾 Golang Slice 的底层实现
{{< details "展开查看详情" >}}
切⽚是基于数组实现的，它的底层是数组，它⾃⼰本身⾮常⼩，可以理解为对底层数组的抽象。因为基于数组实现，所以它的底层的内存是连续分配的，效率⾮常⾼，还可以通过索引获得数据，可以迭代以及垃圾回收优化。切⽚本身并不是动态数组或者数组指针。它内部实现的数据结构通过指针引⽤

底层数组，设定相关属性将数据读写操作限定在指定的区域内。切⽚本身是⼀个只读对象，其⼯作机制类似数组指针的⼀种封装。切⽚对象⾮常⼩，是因为它是只有3 个字段的数据结构：
1. 指向底层数组的指针
2. 切⽚的⻓度
3. 切⽚的容量
{{< /details >}}

#### 🥏 Golang Slice 的扩容机制，有什么注意点？
{{< details "展开查看详情" >}}
**Go 中切⽚扩容的策略是这样的：**
1. ⾸先判断，如果新申请容量⼤于2 倍的旧容量，最终容量就是新申请的容量
2. 否则判断，如果旧切⽚的⻓度⼩于1024，则最终容量就是旧容量的两倍
3. 否则判断，如果旧切⽚⻓度⼤于等于1024，则最终容量从旧容量开始循环增加原来的1/4,直到最终容量⼤于等于新申请的容量
4. 如果最终容量计算值溢出，则最终容量就是新申请容量
{{< /details >}}

#### 🎳 扩容前后的 Slice 是否相同？
{{< details "展开查看详情" >}}
情况⼀：原数组还有容量可以扩容（实际容量没有填充完），这种情况下，扩容以后的数组还是指向原来的数组，对⼀个切⽚的操作可能影响多个指针指向相同地址的 Slice。

情况⼆：原来数组的容量已经达到了最⼤值，再想扩容， Go 默认会先开⼀⽚内存区域，把原来的值拷⻉过来，然后再执⾏ append()操作。这种情况丝毫不影响原数组。

要复制⼀个 Slice，最好使⽤ Copy函数。
{{< /details >}}

#### 🏏 Golang 的参数传递、引⽤类型
{{< details "展开查看详情" >}}
Go 语⾔中所有的传参都是值传递(传值)，都是⼀个副本，⼀个拷⻉。因为拷 ⻉的内容有时候是⾮引⽤类型(int、string、struct 等这些)，这样就在函 数中就⽆法修改原内容数据;有的是引⽤类型(指针、map、slice、chan等 这些)，这样就可以修改原内容数据。

Golang 的引⽤类型包括 slice、map 和 channel。它们有复杂的内部结构，除 了申请内存外，还需要初始化相关属性。内置函数 new 计算类型⼤⼩，为其分 配零值内存，返回指针。⽽ make 会被编译器翻译成具体的创建函数，由其分 配内存和初始化成员结构，返回对象⽽⾮指针。
{{< /details >}}

#### 🏑 Golang Map 底层实现
{{<hide-text hide="答案：*Golang 中 map的底层实现是⼀个散列表，因此实现 map的过程实际上就是实现散表的过程。在这个散列表中，主要出现的结构体有两个，⼀个叫 hmap(a header for a go map)，⼀个叫 bmap(a bucket for a Go map，通常叫其bucket)。*">}}

#### 🏒 new() 与 make() 的区别
{{<hide-text hide="答案： *new只初始化并返回指针，⽽make不仅仅要做初始化，还需要设置⼀些数组的⻓度、容量等*">}}

#### 🏒 Golang Map 如何扩容
{{< details "展开查看详情" >}}
装载因⼦：count/2^B

触发条件：
1. 装填因⼦是否⼤于6.5
2. overflow bucket 是否太多

解决⽅法：
1. 双倍扩容：扩容采取了⼀种称为“渐进式”地⽅式，原有的 key 并不会⼀次性搬迁完毕，每次最多只会搬迁2 个bucket
2. 等量扩容：重新排列，极端情况下，重新排列也解决不了，map成了链表，性能⼤⼤降低，此时哈希种⼦hash0 的设置，可以降低此类极端场景的发⽣。
{{< /details >}}

#### 🥍 Golang Map 查找
{{< details "展开查看详情" >}}
*Go语⾔中 map采⽤的是哈希查找表，由⼀个 key 通过哈希函数得到哈希值，64 位系统中就⽣成⼀个64bit 的哈希值，由这个哈希值将 key 对应到不同的桶*

bucket）中，当有多个哈希映射到相同的的桶中时，使⽤链表解决哈希冲突。key 经过 hash 后共64 位，根据 hmap中 B的值，计算它到底要落在哪个桶时，桶的数量为2^B，如 B=5，那么⽤64 位最后5 位表示第⼏号桶，在⽤ hash 值的⾼8 位确定在 bucket 中的存储位置，当前 bmap中的 bucket 未找到，则查询对应的overflow bucket，对应位置有数据则对⽐完整的哈希值，确定是否是要查找的数据。

如果两个不同的 key 落在的同⼀个桶上，hash 冲突使⽤链表法接近，遍历 bucket 中的 key 如果当前处于 map进⾏了扩容，处于数据搬移状态，则优先从 oldbuckets 查找。
{{< /details >}}

#### 🎯 介绍⼀下 Channel
{{< details "展开查看详情" >}}
Go语⾔中，不要通过共享内存来通信，⽽要通过通信来实现内存共享。Go的 CSP(Communicating SequentialProcess)并发模型，中⽂可以叫做通信顺序进程，是通过 goroutine 和 channel 来实现的。

所以 channel 收发遵循先进先出 FIFO，分为有缓存和⽆缓存，channel 中⼤致有 buffer(当缓冲区⼤⼩部位0 时，是个 ring buffer)、sendx 和 recvx 收发的位置(ring buffer 记录实现)、sendq、recvq 当前 channel 因为缓冲区不⾜⽽阻塞的队列、使⽤双向链表存储、还有⼀个 mutex 锁控制并发、其他原属等。
{{< /details >}}

#### 🪀 Go 语⾔的 Channel 特性？
{{< details "展开查看详情" >}}
1. 给⼀个 nil channel 发送数据，造成永远阻塞
2. 从⼀个 nil channel 接收数据，造成永远阻塞
3. 给⼀个已经关闭的 channel 发送数据，引起 panic
4. 从⼀个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回⼀个零值
5. ⽆缓冲的 channel 是同步的，⽽有缓冲的 channel 是⾮同步的
6. 关闭⼀个 nil channel 将会发⽣ panic
{{< /details >}}

#### 🪁 Channel 的 ring buffer 实现
{{< details "展开查看详情" >}}
channel 中使⽤了 ring buffer(环形缓冲区)来缓存写⼊的数据。ring buffer 有很多好处，⽽且⾮常适合⽤来实现FIFO 式的固定⻓度队列。在 channel 中，ring buffer 的实现如下：
{{< figure src="/posts/Go语言/面试/img/ring_buffer.png" title="" >}}
hchan 中有两个与 buffer 相关的变量:recvx 和 sendx。其中 sendx 表示buffer 中可写的 index，recvx 表示buffer 中可读的 index。从 recvx 到 sendx 之间的元素，表示已正常存放⼊ buffer 中的数据。

我们可以直接使⽤ buf[recvx]来读取到队列的第⼀个元素，使⽤ buf[sendx]= x 来将元素放到队尾。
{{< /details >}}

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E5%90%8E%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E9%9B%8603/  

