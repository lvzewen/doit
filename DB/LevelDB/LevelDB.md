# LevelDB

## 设计思路

做存储的同学都很清楚，对于普通机械磁盘顺序写的性能要比随机写大很多。比如对于15000转的SAS盘，4K写IO， 顺序写在200MB/s左右，而随机写性能可能只有1MB/s左右。而LevelDB的设计思想正是利用了磁盘的这个特性。

LevelDB的数据是存储在磁盘上的，采用LSM-Tree的结构实现。LSM-Tree将磁盘的随机写转化为顺序写，从而大大提高了写速度。为了做到这一点LSM-Tree的思路是将索引树结构拆成一大一小两颗树，较小的一个常驻内存，较大的一个持久化到磁盘，他们共同维护一个有序的key空间。写入操作会首先操作内存中的树，随着内存中树的不断变大，会触发与磁盘中树的归并操作，而归并操作本身仅有顺序写。
