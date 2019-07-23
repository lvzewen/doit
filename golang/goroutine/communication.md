# goroutine并发控制与通信

实现多个goroutine间的同步与通信大致有:

1. 全局共享变量
2. channel通信（CSP模型）
3. Context包

goroutine的退出机制设计是，goroutine退出只能由本身控制，不允许从外部强制结束该goroutine。只有两种情况例外，那就是main函数结束或者程序崩溃结束运行；所以，要实现主进程控制子goroutine的开始和结束，必须借助其它工具来实现。

## 控制并发方式

1. 全局共享变量
2. channel通信（CSP模型）
3. Context包

### 全局共享变量

这是最简单的实现控制并发的方式，实现步骤是：

1. 声明一个全局变量；
2. 所有子goroutine共享这个变量，并不断轮询这个变量检查是否有更新；
3. 在主进程中变更该全局变量；
4. 子goroutine检测到全局变量更新，执行相应的逻辑。

### channel通信

Channel是Go中的一个核心类型，你可以把它看成一个管道，通过它并发核心单元就可以发送或者接收数据进行通讯(communication)。

1. 创建一个Waitgroup的实例wg；
2. 在每个goroutine启动的时候，调用wg.Add(1)注册；
3. 在每个goroutine完成任务后退出之前，调用wg.Done()注销。
4. 在等待所有goroutine的地方调用wg.Wait()阻塞进程，知道所有goroutine都完成任务调用wg.Done()注销之后，Wait()方法会返回。

#### channel

1. 队列存储数据
2. 阻塞和唤醒goroutine

#### select机制

golang 的 select 机制可以理解为是在语言层面实现了和 select, poll, epoll 相似的功能：监听多个描述符的读/写等事件，一旦某个描述符就绪（一般是读或者写事件发生了），就能够将发生的事件通知给关心的应用程序去处理该事件。 golang 的 select 机制是，监听多个channel，每一个 case 是一个事件，可以是读事件也可以是写事件，随机选择一个执行，可以设置default，它的作用是：当监听的多个事件都阻塞住会执行default的逻辑。

channel通信控制基于CSP模型，相比于传统的线程与锁并发模型，避免了大量的加锁解锁的性能消耗，而又比Actor模型更加灵活，使用Actor模型时，负责通讯的媒介与执行单元是紧耦合的–每个Actor都有一个信箱。而使用CSP模型，channel是第一对象，可以被独立地创建，写入和读出数据，更容易进行扩展。

### 杀器Context

Context通常被译作上下文，它是一个比较抽象的概念。在讨论链式调用技术时也经常会提到上下文。一般理解为程序单元的一个运行状态、现场、快照，而翻译中上下又很好地诠释了其本质，上下则是存在上下层的传递，上会把内容传递给下。在Go语言中，程序单元也就指的是Goroutine。
每个Goroutine在执行之前，都要先知道程序当前的执行状态，通常将这些执行状态封装在一个Context变量中，传递给要执行的Goroutine中。上下文则几乎已经成为传递与请求同生存周期变量的标准方法。在网络编程下，当接收到一个网络请求Request，在处理这个Request的goroutine中，可能需要在当前gorutine继续开启多个新的Goroutine来获取数据与逻辑处理（例如访问数据库、RPC服务等），即一个请求Request，会需要多个Goroutine中处理。而这些Goroutine可能需要共享Request的一些信息；同时当Request被取消或者超时的时候，所有从这个Request创建的所有Goroutine也应该被结束。
