# 【摘抄】后端面试题集02


{{< admonition  >}}
本文内容来源于：[【Github】后端面试题集](https://github.com/xiaobaiTech/golangFamily)
{{< /admonition >}}

#### 🎆 说下Go中的锁有哪些?三种锁，读写锁，互斥锁，还有map的安全的锁?
*Go中的三种锁包括:互斥锁,读写锁,sync.Map的安全的锁.*

**互斥锁**

Go并发程序对共享资源进⾏访问控制的主要⼿段，由标准库代码包中sync中的Mutex结构体表示。
```go
//Mutex 是互斥锁， 零值是解锁的互斥锁， ⾸次使⽤后不得复制互斥锁。
type Mutex struct {
 state int32
 sema uint32
}
```
*sync.Mutex包中的类型只有两个公开的指针⽅法Lock和Unlock。*
```go
//Locker表示可以锁定和解锁的对象。
type Locker interface {
 Lock()
 Unlock()
}
//锁定当前的互斥量
//如果锁已被使⽤，则调⽤goroutine
//阻塞直到互斥锁可⽤。
func (m *Mutex) Lock()
//对当前互斥量进⾏解锁
//如果在进⼊解锁时未锁定m，则为运⾏时错误。
//锁定的互斥锁与特定的goroutine⽆关。
//允许⼀个goroutine锁定Mutex然后安排另⼀个goroutine来解锁它。
func (m *Mutex) Unlock()
```
****
**声明⼀个互斥锁：**
```go
var mutex sync.Mutex
```
不像C或Java的锁类⼯具，我们可能会犯⼀个错误：忘记及时解开已被锁住的锁，从⽽导致流程异常。但Go由于存在defer，所以此类问题出现的概率极低。关于defer解锁的⽅式如下：
```go
var mutex sync.Mutex
func Write() {
 mutex.Lock()
 defer mutex.Unlock()
}
```
如果对⼀个已经上锁的对象再次上锁，那么就会导致该锁定操作被阻塞，直到该互斥锁回到被解锁状态.
```go
ort (
 "fmt"
 "sync"
 "time"
)
func main() {
 var mutex sync.Mutex
 fmt.Println("begin lock")
 mutex.Lock()
 fmt.Println("get locked")
 for i := 1; i <= 3; i++ {
 go func(i int) {
 fmt.Println("begin lock ", i)
 mutex.Lock()
 fmt.Println("get locked ", i)
 }(i)
 }
 time.Sleep(time.Second)
 fmt.Println("Unlock the lock")
 mutex.Unlock()
 fmt.Println("get unlocked")
 time.Sleep(time.Second)
}
```
我们在for循环之前开始加锁，然后在每⼀次循环中创建⼀个协程，并对其加锁，但是由于之前已经加锁了，所以这个for循环中的加锁会陷⼊阻塞直到main中的锁被解锁， time.Sleep(time.Second) 是为了能让系统有⾜够的时间运⾏for循环，输出结果如下：
```shell
> go run mutex.go
begin lock
get locked
begin lock 3
begin lock 1
begin lock 2
Unlock the lock
get unlocked
get locked 3
```
这⾥可以看到解锁后，三个协程会重新抢夺互斥锁权，最终协程3获胜。

互斥锁锁定操作的逆操作并不会导致协程阻塞，但是有可能导致引发⼀个⽆法恢复的运⾏时的panic，⽐如对⼀个未锁定的互斥锁进⾏解锁时就会发⽣panic。避免这种情况的最有效⽅式就是使⽤defer。

我们知道如果遇到panic，可以使⽤recover⽅法进⾏恢复，但是如果对重复解锁互斥锁引发的panic却是⽆⽤的（Go 1.8及以后）。
```go
package main
import (
 "fmt"
 "sync"
)
func main() {
 defer func() {
 fmt.Println("Try to recover the panic")
 if p := recover(); p != nil {
 fmt.Println("recover the panic : ", p)
 }
 }()
 var mutex sync.Mutex
 fmt.Println("begin lock")
 mutex.Lock()
 fmt.Println("get locked")
 fmt.Println("unlock lock")
 mutex.Unlock()
 fmt.Println("lock is unlocked")
 fmt.Println("unlock lock again")
 mutex.Unlock()
}
```
**运⾏:**
```shell
> go run mutex.go
begin lock
get locked
unlock lock
l error: sync: unlock of unlocked mutex
goroutine 1 [running]:
runtime.throw(0x4bc1a8, 0x1e)
 /home/keke/soft/go/src/runtime/panic.go:617 +0x72 fp=0xc000084ea8
sp=0xc000084e78 pc=0x427ba2
sync.throw(0x4bc1a8, 0x1e)
 /home/keke/soft/go/src/runtime/panic.go:603 +0x35 fp=0xc000084ec8
sp=0xc000084ea8 pc=0x427b25
sync.(*Mutex).Unlock(0xc00001a0c8)
 /home/keke/soft/go/src/sync/mutex.go:184 +0xc1 fp=0xc000084ef0 sp=0xc000084ec8
pc=0x45f821
main.main()
 /home/keke/go/Test/mutex.go:25 +0x25f fp=0xc000084f98 sp=0xc000084ef0
pc=0x486c1f
runtime.main()
 /home/keke/soft/go/src/runtime/proc.go:200 +0x20c fp=0xc000084fe0
sp=0xc000084f98 pc=0x4294ec
runtime.goexit()
 /home/keke/soft/go/src/runtime/asm_amd64.s:1337 +0x1 fp=0xc000084fe8
sp=0xc000084fe0 pc=0x450ad1
exit status 2
```
这⾥试图对重复解锁引发的panic进⾏recover，但是我们发现操作失败，虽然互斥锁可以被多个协程共享，但还是建议将对同⼀个互斥锁的加锁解锁操作放在同⼀个层次的代码中。

**读写锁**
读写锁是针对读写操作的互斥锁，可以分别针对读操作与写操作进⾏锁定和解锁操作 。

**读写锁的访问控制规则如下：**
1. 多个写操作之间是互斥的
2. 写操作与读操作之间也是互斥的
3. 多个读操作之间不是互斥的

在这样的控制规则下，读写锁可以⼤⼤降低性能损耗。

在Go的标准库代码包中sync中的RWMutex结构体表示为:
```go
// RWMutex是⼀个读/写互斥锁，可以由任意数量的读操作或单个写操作持有。
// RWMutex的零值是未锁定的互斥锁。
//⾸次使⽤后，不得复制RWMutex。
//如果goroutine持有RWMutex进⾏读取⽽另⼀个goroutine可能会调⽤Lock，那么在释放初始读锁之前，
goroutine不应该期望能够获取读锁定。
//特别是，这种禁⽌递归读锁定。 这是为了确保锁最终变得可⽤; 阻⽌的锁定会阻⽌新读操作获取锁定。
type RWMutex struct {
 w Mutex //如果有待处理的写操作就持有
 writerSem uint32 // 写操作等待读操作完成的信号量
 readerSem uint32 //读操作等待写操作完成的信号量
 readerCount int32 // 待处理的读操作数量
 readerWait int32 // number of departing readers
}
```
sync中的RWMutex有以下⼏种⽅法：
```go
//对读操作的锁定
func (rw *RWMutex) RLock()
//对读操作的解锁
func (rw *RWMutex) RUnlock()
//对写操作的锁定
func (rw *RWMutex) Lock()
//对写操作的解锁
func (rw *RWMutex) Unlock()
//返回⼀个实现了sync.Locker接⼝类型的值，实际上是回调rw.RLock and rw.RUnlock.
func (rw *RWMutex) RLocker() Locker
```

Unlock⽅法会试图唤醒所有想进⾏读锁定⽽被阻塞的协程，⽽ RUnlock⽅法只会在已⽆任何读锁定的情况下，试图唤醒⼀个因欲进⾏写锁定⽽被阻塞的协程。若对⼀个未被写锁定的读写锁进⾏写解锁，就会引发⼀个不可恢复的panic，同理对⼀个未被读锁定的读写锁进⾏读写锁也会如此。

由于读写锁控制下的多个读操作之间不是互斥的，因此对于读解锁更容易被忽视。对于同⼀个读写锁，添加多少个读锁定，就必要有等量的读解锁，这样才能其他协程有机会进⾏操作。
```go
package main
import (
 "fmt"
 "sync"
 "time"
)
func main() {
 var rwm sync.RWMutex
 for i := 0; i < 5; i++ {
  go func(i int) {
  fmt.Println("try to lock read ", i)
  rwm.RLock()
  fmt.Println("get locked ", i)
  time.Sleep(time.Second * 2)
  fmt.Println("try to unlock for reading ", i)
  rwm.RUnlock()
  fmt.Println("unlocked for reading ", i)
  }(i)
 }
 time.Sleep(time.Millisecond * 1000)
 fmt.Println("try to lock for writing")
 rwm.Lock()
 fmt.Println("locked for writing")
}
```
**运⾏:**
```shell
> go run rwmutex.go
try to lock read 0
get locked 0
try to lock read 4
get locked 4
try to lock read 3
get locked 3
try to lock read 1
get locked 1
try to lock read 2
get locked 2
try to lock for writing
try to unlock for reading 0
unlocked for reading 0
try to unlock for reading 2
unlocked for reading 2
try to unlock for reading 1
unlocked for reading 1
try to unlock for reading 3
unlocked for reading 3
try to unlock for reading 4
unlocked for reading 4
locked for writing
```
这⾥可以看到创建了五个协程⽤于对读写锁的读锁定与读解锁操作。在 rwm.Lock()种会对main中协程进⾏写锁定，但是for循环中的读解锁尚未完成，因此会造成mian中的协程阻塞。当for循环中的读解锁操作都完成后就会试图唤醒main中阻塞的协程，main中的写锁定才会完成。

**sync.Map安全锁**

golang中的sync.Map是并发安全的，其实也就是sync包中golang⾃定义的⼀个名叫Map的结构体。
```go
package main
import (
 "sync"
 "fmt"
)
func main() {
 //开箱即⽤
 var sm sync.Map
 //store ⽅法,添加元素
 sm.Store(1,"a")
 //Load ⽅法，获得value
 if v,ok:=sm.Load(1);ok{
 fmt.Println(v)
 }
 //LoadOrStore⽅法，获取或者保存
 //参数是⼀对key：value，如果该key存在且没有被标记删除则返回原先的value（不更新）和true；不存在则
store，返回该value 和false
 if vv,ok:=sm.LoadOrStore(1,"c");ok{
 fmt.Println(vv)
 }
 if vv,ok:=sm.LoadOrStore(2,"c");!ok{
 fmt.Println(vv)
 }
 //遍历该map，参数是个函数，该函数参的两个参数是遍历获得的key和value，返回⼀个bool值，当返回false
时，遍历⽴刻结束。
 sm.Range(func(k,v interface{})bool{
 fmt.Print(k)
 fmt.Print(":")
 fmt.Print(v)
 fmt.Println()
 return true
 })
}
```
**运⾏ :**
```shell
a
a
c
1:a
2:c
```
**sync.Map的数据结构:**
```go
type Map struct {
 // 该锁⽤来保护dirty
 mu Mutex
 // 存读的数据，因为是atomic.value类型，只读类型，所以它的读是并发安全的
 read atomic.Value // readOnly
 //包含最新的写⼊的数据，并且在写的时候，会把read 中未被删除的数据拷⻉到该dirty中，因为是普通的map
存在并发安全问题，需要⽤到上⾯的mu字段。
 dirty map[interface{}]*entry
 // 从read读数据的时候，会将该字段+1，当等于len（dirty）的时候，会将dirty拷⻉到read中（从⽽提升
读的性能）。
 misses int
}
```
**read的数据结构是：**
```go
type readOnly struct {
 m map[interface{}]*entry
 // 如果Map.dirty的数据和m 中的数据不⼀样是为true
 amended bool
}
```
**entry的数据结构：**
```go
type entry struct {
 //可⻅value是个指针类型，虽然read和dirty存在冗余情况（amended=false），但是由于是指针类型，存
储的空间应该不是问题
 p unsafe.Pointer // *interface{}
}
```
**Delete ⽅法:**
```go
func (m *Map) Delete(key interface{}) {
 read, _ := m.read.Load().(readOnly)
 e, ok := read.m[key]
 //如果read中没有，并且dirty中有新元素，那么就去dirty中去找
 if !ok && read.amended {
 m.mu.Lock()
 //这是双检查（上⾯的if判断和锁不是⼀个原⼦性操作）
 read, _ = m.read.Load().(readOnly)
 e, ok = read.m[key]
 if !ok && read.amended {
 //直接删除
 delete(m.dirty, key)
 }
 m.mu.Unlock()
 }
 if ok {
//如果read中存在该key，则将该value 赋值nil（采⽤标记的⽅式删除！）
 e.delete()
 }
}
func (e *entry) delete() (hadValue bool) {
 for {
  p := atomic.LoadPointer(&e.p)
  if p == nil || p == expunged {
  return false
  }
  if atomic.CompareAndSwapPointer(&e.p, p, nil) {
  return true
  }
 }
}
```
**Store ⽅法:**
```go
func (m *Map) Store(key, value interface{}) {
  // 如果m.read存在这个key，并且没有被标记删除，则尝试更新。
  read, _ := m.read.Load().(readOnly)
  if e, ok := read.m[key]; ok && e.tryStore(&value) {
  return
 }
 // 如果read不存在或者已经被标记删除
 m.mu.Lock()
 read, _ = m.read.Load().(readOnly)
 if e, ok := read.m[key]; ok {
  //如果entry被标记expunge，则表明dirty没有key，可添加⼊dirty，并更新entry
  if e.unexpungeLocked() {
  //加⼊dirty中
  m.dirty[key] = e
  }
  //更新value值
  e.storeLocked(&value)
 //dirty 存在该key，更新
 } else if e, ok := m.dirty[key]; ok {
  e.storeLocked(&value)
 //read 和dirty都没有，新添加⼀条
 } else {
  //dirty中没有新的数据，往dirty中增加第⼀个新键
  if !read.amended {
  //将read中未删除的数据加⼊到dirty中
  m.dirtyLocked()
  m.read.Store(readOnly{m: read.m, amended: true})
  }
  m.dirty[key] = newEntry(value)
 }
}
//将read中未删除的数据加⼊到dirty中
func (m *Map) dirtyLocked() {
 if m.dirty != nil {
  return
 }
 read, _ := m.read.Load().(readOnly)
 m.dirty = make(map[interface{}]*entry, len(read.m))
 //read如果较⼤的话，可能影响性能
 for k, e := range read.m {
  //通过此次操作，dirty中的元素都是未被删除的，可⻅expunge的元素不在dirty中
  if !e.tryExpungeLocked() {
  m.dirty[k] = e
  }
 }
}
//判断entry是否被标记删除，并且将标记为nil的entry更新标记为expunge
func (e *entry) tryExpungeLocked() (isExpunged bool) {
 p := atomic.LoadPointer(&e.p)
 for p == nil {
 // 将已经删除标记为nil的数据标记为expunged
 if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
 return true
 }
 p = atomic.LoadPointer(&e.p)
 }
 return p == expunged
}
//对entry 尝试更新
func (e *entry) tryStore(i *interface{}) bool {
  p := atomic.LoadPointer(&e.p)
  if p == expunged {
  return false
 }
 for {
  if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
  return true
  }
  p = atomic.LoadPointer(&e.p)
  if p == expunged {
  return false
  }
 }
}
//read⾥ 将标记为expunge的更新为nil
func (e *entry) unexpungeLocked() (wasExpunged bool) {
 return atomic.CompareAndSwapPointer(&e.p, expunged, nil)
}
//更新entry
func (e *entry) storeLocked(i *interface{}) {
 atomic.StorePointer(&e.p, unsafe.Pointer(i))
}
```
因此，每次操作先检查read，因为read 并发安全，性能好些；read不满⾜，则加锁检查dirty，⼀旦是新的键值，dirty会被read更新。

Load⽅法:

Load⽅法是⼀个加载⽅法，查找key。
```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
 //因read只读，线程安全，先查看是否满⾜条件
 read, _ := m.read.Load().(readOnly)
 e, ok := read.m[key]
 //如果read没有，并且dirty有新数据，那从dirty中查找，由于dirty是普通map，线程不安全，这个时候⽤
到互斥锁了
 if !ok && read.amended {
 m.mu.Lock()
 // 双重检查
 read, _ = m.read.Load().(readOnly)
 e, ok = read.m[key]
 // 如果read中还是不存在，并且dirty中有新数据
 if !ok && read.amended {
 e, ok = m.dirty[key]
 // mssLocked（）函数是性能是sync.Map 性能得以保证的重要函数，⽬的讲有锁的dirty数据，
替换到只读线程安全的read⾥
 m.missLocked()
 }
 m.mu.Unlock()
 }
 if !ok {
 return nil, false
 }
 return e.load()
}
//dirty 提升⾄read 关键函数，当misses 经过多次因为load之后，⼤⼩等于len（dirty）时候，讲dirty替换
到read⾥，以此达到性能提升。
func (m *Map) missLocked() {
 m.misses++
 if m.misses < len(m.dirty) {
 return
 }
//原⼦操作，耗时很⼩
 m.read.Store(readOnly{m: m.dirty})
 m.dirty = nil
 m.misses = 0
}
```
*sync.Map是通过冗余的两个数据结构(read、dirty),实现性能的提升。为了提升性能，load、delete、store等操作尽量使⽤只读的read；为了提⾼read的key击中概率，采⽤动态调整，将dirty数据提升为read；对于数据的删除，采⽤延迟标记删除法，只有在提升dirty的时候才删除。*

---

> 作者: [Kendrick](https://kendrickcheung.github.io/)  
> URL: https://kendrickcheung.github.io/%E5%90%8E%E7%AB%AF%E9%9D%A2%E8%AF%95%E9%A2%98%E9%9B%8602/  

