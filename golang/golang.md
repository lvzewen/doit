1. go struct能不能比较
因为是强类型语言，所以不同类型的结构不能作比较，但是同一类型的实例值是可以比较的，实例不可以比较，因为是指针类型
2. go defer（for defer），先进后出，后进先出
3. select可以用于什么，常用于gorotine的完美退出
golang 的 select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作每个case语句里必须是一个IO操作，确切的说，应该是一个面向channel的IO操作
4. context包的用途Context通常被译作上下文，它是一个比较抽象的概念，其本质，是【上下上下】存在上下层的传递，上会把内容传递给下。在Go语言中，程序单元也就指的是Goroutine
5. client如何实现长连接
server是设置超时时间，for循环遍历的
6. 主协程如何等其余协程完再操作
使用channel进行通信，context,select
7. slice，len，cap，共享，扩容
append函数，因为slice底层数据结构是，由数组、len、cap组成，所以，在使用append扩容时，会查看数组后面有没有连续内存快，有就在后面添加，没有就重新生成一个大的数组。
8. map如何顺序读取
map不能顺序读取，是因为他是无序的，想要有序读取，首先的解决的问题就是，把ｋｅｙ变为有序，所以可以把key放入切片，对切片进行排序，遍历切片，通过key取值。
9. 实现消息队列（多生产者，多消费者）
使用切片加锁可以实现
10. 大文件排序
归并排序，分而治之,拆分为小文件，在排序
11. 基本排序，哪些是稳定的
14、http get跟head
1HEAD和GET本质是一样的，区别在于HEAD不含有呈现数据，而仅仅是HTTP头信息。有的人可能觉得这个方法没什么用，其实不是这样的。想象一个业务情景：欲判断某个资源是否存在，我们通常使用GET，但这里用HEAD则意义更加明确。
15、http 401,403
400 bad request，请求报文存在语法错误
401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
403 forbidden，表示对请求资源的访问被服务器拒绝
404 not found，表示在服务器上没有找到请求的资源
16、http keep-alive
client发出的HTTP请求头需要增加Connection:keep-alive字段
Web-Server端要能识别Connection:keep-alive字段，并且在http的response里指定Connection:keep-alive字段，告诉client，我能提供keep-alive服务，并且"应允"client我暂时不会关闭socket连接
17、http能不能一次连接多次请求，不等后端返回
http本质上使用socket连接，因此发送请求，接写入tcp缓冲，是可以多次进行的，这也是http是无状态的原因
18、tcp与udp区别，udp优点，适用场景
tcp传输的是数据流，而udp是数据包，tcp会进过三次握手，udp不需要
19、time-wait的作用
20、数据库如何建索引
21、孤儿进程，僵尸进程
22、死锁条件，如何避免
23、linux命令，查看端口占用，cpu负载，内存占用，如何发送信号给一个进程
24、git文件版本，使用顺序，merge跟rebase
25、Slice与数组区别，Slice底层结构