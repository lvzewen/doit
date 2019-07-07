# 安全多方计算(MPC)

安全计算，中文全称为安全多方计算，英文全称为Secure Multi-party Computation，缩写为SMC或 MPC，指的是在保护数据安全的前提下实现多方计算。

## 多方计算

多个参与者将各自的数据凑在一起，并在这个大数据集上进行一定的计算，并得到最后的计算结果。形式化的描述就是：假定有 [公式] 个参与方 [公式] ，他们各自拥有自己的数据集 [公式] 。那么多方计算的目的就是得到某个函数 [公式] 的输出，该函数的输入就是上面的 [公式] 们，即 [公式] 。举个简单的例子：假定五道口某工科学校想统计该校男生的恋爱经历分布情况，那 [公式] 取值就是0（母胎单）或1（被甩过）。在这种情况下f就是简单的求和，即 [公式] 。

主要计算需求：

1.比较问题

2.线性操作（加乘）

3.非线性操作（sigmod函数，relu函数）

4.求交集

安全计算实现方法概览 <https://zhuanlan.zhihu.com/p/31641175>

大致可归为两类：一类是基于噪音的，另一类不是基于噪音的

基于噪音的安全计算方法，最主要代表是目前很火的差分隐私（differential privacy）。这类方法的思想是，对计算过程用噪音干扰，让原始数据淹没在噪音中，使别有用心者无法从得到的结果反推原始数据。这就好像我们拿到一张打了马赛克的图片，虽然可能可以猜出马赛克后面大概长啥样，但很难知道马赛克后面的所有细节。
这个干扰既可以是数据源，也可以是模型参数和输出。

非噪音方法一般是通过密码学方法将数据编码或加密，得到一些奇怪的数字，而且这些奇怪的数字有一些神奇的性质，比如看上去很随机但其实保留了原始数据的线性关系，或者顺序明明被打乱但人们却能从中很容易找到与原始数据的映射关系。
这一类方法主要包括三种：混淆电路（Garbled Circuit）、同态加密（Homomorphic Encryption）和密钥分享（Secret Sharing）。这些方法一般是在源头上就把数据加密或编码了，计算操作方看到的都是密文，因此只要特定的假设条件满足，这类方法在计算过程中是不会泄露信息的。

相比于前一类基于噪音的方法，这种方法的优点是不会对计算过程加干扰，因此我们最终得到的是准确值，且有密码学理论加持，安全性有保障，缺点则是由于使用了很多密码学方法，整个过程中无论是计算量还是通讯量都非常庞大，对于一些复杂的任务（如训练几十上百层的CNN等），短时间内可能无法完成

同态加密
如果我们有一个加密函数 f , 把明文A变成密文A’, 把明文B变成密文B’，也就是说 f(A) = A’ ， f(B) = B’ 。另外我们还有一个解密函数 f−1f−1 能够将 f 加密后的密文解密成加密前的明文。

对于一般的加密函数，如果我们将A’和B’相加，得到C’。我们用f−1f−1 对C’进行解密得到的结果一般是毫无意义的乱码。

但是，如果 f 是个可以进行同态加密的加密函数， 我们对C’使用 f−1f−1 进行解密得到结果C， 这时候的C = A + B。这样，数据处理权与数据所有权可以分离，这样企业可以防止自身数据泄露的同时，利用云服务的算力。

同态分类

a) 如果满足 f(A)+f(B)=f(A+B)f(A)+f(B)=f(A+B) ， 我们将这种加密函数叫做加法同态

b) 如果满足 f(A)×f(B)=f(A×B)f(A)×f(B)=f(A×B) ，我们将这种加密函数叫做乘法同态。

如果一个加密函数f只满足加法同态，就只能进行加减法运算；

如果一个加密函数f只满足乘法同态，就只能进行乘除法运算;

如果一个加密函数同时满足加法同态和乘法同态，称为全同态加密。那么这个使用这个加密函数完成各种加密后的运算(加减乘除、多项式求值、指数、对数、三角函数)。

同态加密的实现原理是什么？在实际中有何应用？

<https://www.zhihu.com/question/27645858>

Fully Homomorphic Encryption (FHE)：这意味着HE方案支持任意给定的f函数，只要这个f函数可以通过算法描述，用计算机实现。显然，FHE方案是一个非常棒的方案，但是计算开销极大，暂时还无法在实际中使用。
Somewhat Homomorphic Encryption (SWHE)：这意味着HE方案只支持一些特定的f函数。SWHE方案稍弱，但也意味着开销会变得较小，容易实现，现在已经可以在实际中使用。（最最经典的RSA加密，乘法运算）
公钥加密方案有三个算法：KeyGen算法（密钥生成），Enc算法（加密），Dec算法（解密），Evaluate算法（密文计算）

B.差分隐私Differential Privacy https://zhuanlan.zhihu.com/p/40760105

保护的是数据源中一点微小的改动导致的隐私泄露问题。比如有一群人出去聚餐，那么其中某人是否是单身狗就属于差分隐私。

## 安全多方计算的价值

MPC是密码学的一个重要分支，旨在解决一组互不信任的参与方之间保护隐私的协同计算问题，为数据需求方提供不泄露原始数据前提下的多方协同计算能力。
　　在目前个人数据毫无隐私的环境下，对数据进行确权并实现数据价值显得尤为重要。MPC就是实现此目的的计算协议，在整个计算协议执行过程中，用户对个人数据始终拥有控制权，只有计算逻辑是公开的。计算参与方只需参与计算协议，无需依赖第三方就能完成数据计算，并且参与各方拿到计算结果后也无法推断出原始数据。

## 安全多方计算的来源

安全多方计算（MPC：Secure Muti-Party Computation）研究由图灵奖获得者、×××院士姚期智教授在1982年提出，姚教授以著名的百万富翁问题来说明安全多方计算。百万富翁问题指的是，在没有可信第三方的前提下，两个百万富翁如何不泄露自己的真实财产状况来比较谁更有钱。通过研究此问题，形象地说明了安全多方计算面临的挑战和问题解决思路，经Oded Goldreich、Shaft Goldwasser等学者的众多原始创新工作，安全多方计算逐渐发展成为密码学的一个重要分支。

## 问题抽象

安全多方计算可以抽象的理解为：两方分别拥有各自的私有数据，在不泄漏各自私有数据的情况下，能够计算出关于公共函数 的结果。整个计算完成时，只有计算结果对双方可知，且双方均不知对方的数据以及计算过程的中间数据。

## 什么是安全多方计算

多个持有各自私有数据的参与方，共同执行一个计算逻辑计算逻辑（如，求最大值计算），并获得计算结果。但过程中，参与的每一方均不会泄漏各自数据的计算，被称之为MPC计算。
　　举个例子，Bob和Alice想弄清谁的薪资更高，但因为签署了保密协议而不能透露具体薪资。如果Bob和Alice分别将各自的薪资告诉离职员工Anne，这时Anne就能知道谁的薪资更高，并告诉Bob和Alice。这种方式就是需保证中间人Anne完全可信。
　　而通过MPC则可以设计一个协议，在这个协议中，算法取代中间人的角色，Alice和Bob的薪资以及比较的逻辑均交由算法处理，参与方只需执行计算协议，而不用依赖于一个完全可信的第三方。
　　安全多方计算所要确保的基本性质就是：在协议执行期间发送的消息中不能推断出各方持有的私有数据信息，关于私有数据唯一可以推断的信息是仅仅能从输出结果得到的信息。

## 什么是算法

算法（Algorithm）是指解题方案的准确而完整的描述，是一系列解决问题的清晰指令，算法代表着用系统的方法描述解决问题的策略机制。也就是说，能够对一定规范的输入，在有限时间内获得所要求的输出。
　　如果一个算法有缺陷，或不适合于某个问题，执行这个算法将不会解决这个问题。不同的算法可能用不同的时间、空间或效率来完成同样的任务。一个算法的优劣可以用空间复杂度与时间复杂度来衡量。
　　算法具有以下五个重要特征：

    穷性：算法的有穷性是指算法必须能在执行有限个步骤之后终止；
    确切性：算法的每一步骤必须有确切的定义；
    输入项：一个算法有0个或多个输入，以刻画运算对象的初始情况，所谓0个输入是指算法本身定出了初始条件；
    输出项：一个算法有一个或多个输出，以反映对输入数据加工后的结果。没有输出的算法是毫无意义的；
    可行性：算法中执行的任何计算步骤都是可以被分解为基本的可执行的操作步，即每个计算步都可以在有限时间内完成（也称之为有效性）。

## MPC问题分类

由算法适用性来看，MPC既适用于特定的算法，如加法、乘法、AES，集合交集等；也适用于所有可表示成计算过程的通用算法。
根据计算参与方个数不同，可分为只有两个参与方的2PC和多个参与方（≥3）的通用MPC。
安全两方计算所使用的协议为Garbled Circuit(GC)+Oblivious Transfer(OT)；而安全多方计算所使用的协议为同态加密+秘密分享+OT。
在安全多方计算中，安全挑战模型包括半诚实敌手模型和恶意敌手模型。市场大部分场景满足半诚实敌手模型，也是JUGO技术产品所考虑的敌手模型。
半诚实敌手模型：计算方存在获取其他计算方原始数据的需求，但仍按照计算协议执行。半诚实关系即参与方之间有一定的信任关系，适合机构之间的数据计算；
恶意敌手模型：参与方根本就不按照计算协议执行计算过程。参与方可采用任何（恶意）方式与对方通信，且没有任何信任关系。结果可能是协议执行不成功，双方得不到任何数据；或者协议执行成功，双方仅知道计算结果。更多适用于个人之间、或者个人与机构之间的数据计算。`https://blog.51cto.com/13701316/2136084`