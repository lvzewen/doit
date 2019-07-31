# 以太坊要点

Gas是Ethereum系统的血液。一切资源，活动，交互的开销，都以Gas为计量单元。如果定义了一个GasPrice，那么所有的Gas消耗亦可等价于以太币Ether。
Block是Transaction的集合。Block在插入BlockChain前，需要将所有Transaction逐个执行。Transaction的执行会消耗发起方的Ether，但系统在其执行完成时，会给予其作者（挖掘出这个Block的账户）一笔补偿，这笔补偿是“矿工”赚取收入的来源之一。
Ethereum 定义了自己的虚拟机EVM, 它与合约(Contract)机制相结合，能够在提供非常丰富的操作的同时，又能很好的控制存储空间和运行速度。Contract由Transaction转化得到。
Ethereum 里的哈希函数，用的是SHA-3，256 bits；数据(数组)的序列化，用的是RLP编码，所以所有对象，数组的哈希算法，实际用的RLP + SHA-3。数字签名算法，使用了椭圆曲线数字签名算法(ECDSA)。

Block结构体主要分为Header和Body，Header相对轻量，涵盖了Block的所有属性，包括特征标示，前向指针，和内部数据集的验证哈希值等；Body相对重量，持有内部数据集。每个Block的Header部分，Body部分，以及一些特征属性，都以[k,v]形式单独存储在底层数据库中。
BlockChain管理Block组成的一个单向链表，HeaderChain管理Header组成的单向链表，并且BlockChain持有HeaderChain。在做插入/删除/查找时，要注意回溯，以及数据库中相应的增删。
Merkle-PatriciaTrie(MPT)数据结构用来组织管理[k,v]型数据，它设计了灵活多变的节点体系和编码格式，既融合了多种原型结构的优点，又兼顾了业务需求和运行效率。
StateDB作为本地存储模块，它面向业务模型，又连接底层数据库，内部利用两极缓存机制来存储和更新所有代表“账户”的stateObject对象。
stateObject除了管理着账户余额等信息之外，也用了类似的两级缓存机制来存储和更新所有的State数据。
一般所说的“挖掘一个新区块”其实包括两部分，第一阶段组装出新区块的所有数据成员，包括交易列表txs、叔区块uncles等，并且所有交易都已经执行完毕，各帐号状态更新完毕；第二阶段对该区块进行授勋/封印(Seal)，没有成功Seal的区块不能被广播给其他节点。第二阶段所消耗的运算资源，远超第一阶段。
Seal过程由共识算法(consensus algorithm)族完成，包括Ethash算法和Clique算法两种实现。前者是产品环境下真实采用的，后者是针对测试网络(testnet)使用的。Seal()函数并不会增加或修改区块中任何跟有效数据有关的部分，它的目的是通过一系列复杂的步骤，或计算或公认，选拔出能够出产新区块的个体。
Ethash算法(PoW)基于运算能力来筛选出挖掘区块的获胜者，运算过程中使用了大量、多次、多种的哈希函数，通过极高的计算资源消耗，来限制某些节点通过超常规的计算能力轻易形成“中心化”倾向。
Clique算法(PoA)利用数字签名算法完成Seal操作，不过签名所用公钥，同时也是common.Address类型的地址必须是已认证的。所有认证地址基于特殊的投票地址进行动态管理，记名投票由不记名投票和投票方地址随机组合而成，杜绝重复的不记名投票，严格限制外部代码恶意操纵投票数据
在实践“去中心化”方面，以太坊还在区块结构中增加了叔区块(uncles)成员以加大计算资源的消耗，并通过在交易执行环节对叔区块作者(挖掘者)的奖励，以收益机制来调动网络中各节点运算资源分布更加均匀。
eth.ProtocolManager中，会对每一个远端peer发起主动传输数据的操作，这组操作按照数据类型区分，可分为交易和区块；而若以发送数据方式来区分，亦可分为广播单项数据，和同步一组同类型数据。这样两两配对，即可形成4组主动传输数据的操作。
ProtocolManager通过在p2p.Protocol{}对象中埋入回调函数，可以对远端peer的任何事件及状态更新作出响应。这些Protocol对象，会由p2p.Server传递给每一个新连接上的远端peer。
以太坊目前实现的p2p通信协议族的结构类型中，按照功能和作用，可分为三层：顶层pkg eth中的类型直接服务于当前以太坊系统（Ethereum，ProtocolManager等模块），中间层pkg p2p是泛化结构类型，底层包括golang语言包自带的pkg net， syscall等，封装了网络层和传输层协议的系统实现。
以太坊中代码中，accounts.Manager是管理账户信息的模块。Manager可以管理多个<Wallet>的实现，每个<Wallet>实现拥有多个Account账户，每个Account对应一个Address地址，而以太币Ether存放于每个Address上。以太坊同时提供软件版和硬件版的<Wallet>实现。
以太坊中，每个Address类型变量均来自于椭圆曲线数字签名算法(ECDSA)所用的公钥，因此钱包程序还必须提供管理数字签名公钥密钥的功能。软件版accounts.<Wallet>实现叫keystore，通过在本地文件系统中分别显式存储账户信息和加密存储公钥密钥的方式，提供以上功能。
以太坊客户端程序之间，会通过accounts.Manager模块相互订阅Wallet更新事件，以保证每个客户端个体(peer)，都能及时更新全网络中的完整Wallet列表。
客户端程序的核心是eth.Ethereum，它以RPC service的形式，向外提供内部各模块的功能，诸如挖掘区块, 数据库读写，p2p下载等。