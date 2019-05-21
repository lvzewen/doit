# panic

## panic解释

Golang里比较常见的错误处理方法是返回error给调用者，但如果是无法恢复的错误，返回error也没有意义，此时可以选择go die：主动触发panic。

除了代码中主动触发的panic，程序运行过程中也会因为出现某些错误而触发panic，例如数组越界。

panic会停掉当前正在执行的程序（注意，不只是协程），但是与os.Exit(-1)这种直愣愣的退出不同，panic的撤退比较有秩序，他会先处理完当前goroutine已经defer挂上去的任务，执行完毕后再退出整个程序。

而defer的存在，让我们有更多的选择，比如在defer中通过recover截取panic，从而达到try..catch的效果。

panic允许传递一个参数给他，参数通常是将出错的信息以字符串的形式来表示。panic会打印这个字符串，以及触发panic的调用栈。

`执行，且只执行，当前goroutine的defer。`
如果当前函数中有多个defer呢？

defer的特点就是LIFO，即后进先出，所以如果在同一个函数下多个defer的话，会逆序执行。

那调用者的defer会被执行吗？

panic仅保证当前goroutine下的defer都会被调到，但不保证其他协程的defer也会调到。如果是在同一goroutine下的调用者的defer，那么可以一路回溯回去执行；但如果是不同goroutine，那就不做保证了。

总结：
遇到处理不了的错误，找panic
panic有操守，退出前会执行本goroutine的defer，方式是原路返回(reverse order)
panic不多管，不是本goroutine的defer，不执行

## recover

有时我们不希望因为无法处理错误panic而导致整个进程挂掉，因此需要像java一样能够handle panic（异常处理机制）。

golang在这种情况下可以在panic的当前goroutine的defer中使用recover来捕捉panic。

注意recover只在defer的函数中有效，如果不是在refer上下文中调用，recover会直接返回nil。

## defer

defer是一个面向编译器的声明，他会让编译器做两件事：

1. 编译器会将defer声明编译为runtime.deferproc(fn)，这样运行时，会调用runtime.deferproc，在deferproc中将所有defer挂到goroutine的defer链上；

2. 编译器会在函数return之前（注意，是return之前，而不是return xxx之前，后者不是一条原子指令），增加runtime.deferreturn调用；这样运行时，开始处理前面挂在defer链上的所有defer。

`https://ieevee.com/tech/2017/11/23/go-panic.html#panic`