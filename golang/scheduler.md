# Go runtime调度

## goroutine简介

goroutine是go语言中最为NB的设计，也是其魅力所在，goroutine的本质是协程，是实现并行计算的核心。goroutine使用方式非常的简单，只需使用go关键字即可启动一个协程，并且它是处于异步方式运行，你不需要等它运行完成以后在执行以后的代码。

## goroutine内部原理

### 概念介绍

1. 并发
一个cpu上能同时执行多项任务，在很短时间内，cpu来回切换任务执行(在某段很短时间内执行程序a，然后又迅速得切换到程序b去执行)，有时间上的重叠（宏观上是同时的，微观仍是顺序执行）,这样看起来多个任务像是同时执行，这就是并发。
2. 并行
当系统有多个CPU时,每个CPU同一时刻都运行任务，互不抢占自己所在的CPU资源，同时进行，称为并行。
3. 进程
cpu在切换程序的时候，如果不保存上一个程序的状态（也就是我们常说的context--上下文），直接切换下一个程序，就会丢失上一个程序的一系列状态，于是引入了进程这个概念，用以划分好程序运行时所需要的资源。因此进程就是一个程序运行时候的所需要的基本资源单位（也可以说是程序运行的一个实体）。
4. 线程
cpu切换多个进程的时候，会花费不少的时间，因为切换进程需要切换到内核态，而每次调度需要内核态都需要读取用户态的数据，进程一旦多起来，cpu调度会消耗一大堆资源，因此引入了线程的概念，线程本身几乎不占有资源，他们共享进程里的资源，内核调度起来不会那么像进程切换那么耗费资源。
5. 协程
协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此，协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作执行者则是用户自身程序，goroutine也是协程。

### 调度模型简介

groutine能拥有强大的并发实现是通过GPM调度模型实现，下面就来解释下goroutine的调度模型。

Go的调度器内部有四个重要的结构：M，P，S，Sched，如上图所示（Sched未给出）
M:M代表内核级线程，一个M就是一个线程，goroutine就是跑在M之上的；M是一个很大的结构，里面维护小对象内存cache（mcache）、当前执行的goroutine、随机数发生器等等非常多的信息
G:代表一个goroutine，它有自己的栈，instruction pointer和其他信息（正在等待的channel等等），用于调度。
P:P全称是Processor，处理器，它的主要用途就是用来执行goroutine的，所以它也维护了一个goroutine队列，里面存储了所有需要它来执行的goroutine。
Sched：代表调度器，它维护有存储M和G的队列以及调度器的一些状态信息等。

### 调度实现

## 使用goroutine

### 基本使用

设置goroutine运行的CPU数量，最新版本的go已经默认已经设置了。
num := runtime.NumCPU()    //获取主机的逻辑CPU个数runtime.GOMAXPROCS(num)    //设置可同时执行的最大CPU数

### goroutine异常捕捉

当启动多个goroutine时，如果其中一个goroutine异常了，并且我们并没有对进行异常处理，那么整个程序都会终止，所以我们在编写程序时候最好每个goroutine所运行的函数都做异常处理，异常处理采用recover

## channel

### 简介

channel俗称管道，用于数据传递或数据共享，其本质是一个先进先出的队列，使用goroutine+channel进行数据通讯简单高效，同时也线程安全，多个goroutine可同时修改一个channel，不需要加锁。

channel可分为三种类型：
只读channel：只能读channel里面数据，不可写入
只写channel：只能写数据，不可读
一般channel：可读可写

读写数据
需要注意的是：
管道如果未关闭，在读取超时会则会引发deadlock异常
管道如果关闭进行写入数据会pannic
当管道中没有数据时候再行读取或读取到默认值，如int类型默认值是0

循环管道
需要注意的是：
使用range循环管道，如果管道未关闭会引发deadlock错误。
如果采用for死循环已经关闭的管道，当管道没有数据时候，读取的数据会是管道的默认值，并且循环不会退出。

只读channel和只写channel
一般定义只读和只写的管道意义不大，更多时候我们可以在参数传递时候指明管道可读还是可写，即使当前管道是可读写的。

select-case实现非阻塞channel
原理通过select+case加入一组管道，当满足（这里说的满足意思是有数据可读或者可写)select中的某个case时候，那么该case返回，若都不满足case，则走default分支。

channel频率控制
在对channel进行读写的时，go还提供了非常人性化的操作，那就是对读写的频率控制，通过time.Ticke实现

`http://www.imooc.com/article/42071`