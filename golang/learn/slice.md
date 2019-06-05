# 切片(Slice)

## 声明一个数组

```golang
var array [5]int //声明一个数组。
var array2 [5]int
array2=array //两个数组变量赋值，这个操作会导致一次数组的拷贝。
```

## 声明一个数组指针

```golang
var array [5]*int //声明一个数组指针。
var array2 [5]*int
array2=array //两个数组指针赋值，不会拷贝底层数组，两个指针都指向同一个数组。
```

## Slice

slice并不是单纯的一个指向数组的指针，它是一个结构体（包含：指针，长度，容量）。golang中的nil也不是一个单纯的空指针。

```golang
// 创建 nil 整型切片
var slice []int
```

`var slice []int=nil`
表示slice被一个nil切片赋值，nil切片就是一个结构体中个变量都为"0值"。
即：nil切片{指针（NULL），长度（0），容量（0）}

## Slice的赋值

如果讲一个Slice赋值个另外一个Slice了，那么他们是共享底层数组的
比如slice2=slice
那么，slice2中的指针与slice中的指针就指向了底层同一个数组。

还可以通过索引的方式来创建切片：

```golang
// 创建一个整型切片
// 其长度和容量都是 5 个元素
slice := []int{10, 20, 30, 40, 50}
// 创建一个新切片
// 其长度为 2 个元素，容量为 4 个元素
newSlice := slice[1:3]
/ 将第三个元素切片，并限制容量
// 其长度为 1 个元素，容量为 2 个元素
newSlice2 := slice[2:3:4]
```

Slice中长度代表了他能访问的区间，容量代表了append操作的区间。
如果append超过设定的区间，那么Slice底层就会扩容了

## apped 陷阱

`append操作可能会导致原本使用同一个底层数组的两个Slice变量变为使用不同的底层数组。`

```golang
func test(){
     var array =[]int{1,2,3,4,5}// len:5,capacity:5
     var newArray=array[1:3]// len:2,capacity:4   (已经使用了两个位置，所以还空两位置可以append)
     fmt.Printf("%p\n",array) //0xc420098000
     fmt.Printf("%p\n",newArray) //0xc420098008 可以看到newArray的地址指向的是array[1]的地址，即他们底层使用的还是一个数组
     fmt.Printf("%v\n",array) //[1 2 3 4 5]
     fmt.Printf("%v\n",newArray) //[2 3]

     newArray[1]=9 //更改后array、newArray都改变了
     fmt.Printf("%v\n",array) // [1 2 9 4 5]
     fmt.Printf("%v\n",newArray) // [2 9]

     newArray=append(newArray,11,12)//append 操作之后，array的len和capacity不变,newArray的len变为4，capacity：4。因为这是对newArray的操作
     fmt.Printf("%v\n",array) //[1 2 9 11 12] //注意对newArray做append操作之后，array[3],array[4]的值也发生了改变
     fmt.Printf("%v\n",newArray) //[2 9 11 12]

     newArray=append(newArray,13,14) // 因为newArray的len已经等于capacity，所以再次append就会超过capacity值，
     // 此时，append函数内部会创建一个新的底层数组（是一个扩容过的数组），并将array指向的底层数组拷贝过去，然后在追加新的值。
     fmt.Printf("%p\n",array) //0xc420098000
     fmt.Printf("%p\n",newArray) //0xc4200a0000
     fmt.Printf("%v\n",array) //[1 2 9 11 12]
     fmt.Printf("%v\n",newArray) //[2 9 11 12 13 14]  他两已经不再是指向同一个底层数组y了
}
```

## 切片的拷贝

使用 Go 语言内建的 copy() 函数，可以迅速地将一个切片的数据复制到另外一个切片空间中，copy() 函数的使用格式如下：
`copy( destSlice, srcSlice []T) int`

srcSlice 为数据来源切片。
destSlice 为复制的目标。目标切片必须分配过空间且足够承载复制的元素个数。来源和目标的类型一致，copy 的返回值表示实际发生复制的元素个数。

