# 并行拜占庭容错算法(Concurrent Byzantine Fault Tolerance)

CBFT算法有四个阶段：block determination、pre-prepare、prepare 和 commit，后三个阶段与PBFT算法三个阶段类似。CBFT的一个重要优势是并发性，每个块可以与其他块并发的方式投及建块，从而大大的提高共识速度。CBFT另一个重要特点就是可以在提交阶段检测受节点，可以在最后阶段广播消息来识别叛徒节点。步骤包含：交易级别的确认和投票、块、块验证、块确认。
