# Counter-Examples: 跨书 CS 核心思维中的失败模式与陷阱

> 提取自 8 本经典教材中作者明确警告的 fallacies, pitfalls, 反例与边界条件。
> 每条反例均绑定到对应的正面 skill，作为其 Boundary (B) 字段的候选来源。

---

## 体系结构 / 量化分析

- id: ce01
  title: 缓存越大越好
  type: counter-example
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Fallacies and Pitfalls"
  source_quote: |
    "Bigger caches are always better."
  failure_mode: |
    盲目增大缓存容量导致命中时间（hit time）显著增加，反而降低处理器时钟频率或增加流水线级数，最终整体性能下降。
  mechanism: |
    缓存访问延迟与容量呈非线性增长（通常与平方根或立方根相关）。当容量超过工作集后，边际命中率收益递减，而访问延迟的固定成本持续累积。Amdahl 定律表明：若命中时间占关键路径比例过高，即使命中率微增也无法抵消周期时间的损失。
  warning_signs:
    - "缓存容量翻倍但 IPC (Instructions Per Cycle) 未提升甚至下降"
    - "处理器最大时钟频率因缓存级数增加而被拖累"
    - "工作集大小远小于实际缓存容量"
  bound_to:
    - "局部性驱动的优化决策"
    - "定量性能分析框架"
  tags: [caching, locality, diminishing-returns, latency, quantitative-analysis]

- id: ce02
  title: 流水线深度可以无限增加
  type: counter-example
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Fallacies and Pitfalls"
  source_quote: |
    "Pipelines can be infinitely deep."
  failure_mode: |
    过度细分流水线级数导致分支预测失败和缓存未命中的惩罚代价呈指数级放大，实际 CPI 恶化，功耗也急剧上升。
  mechanism: |
    每增加一级流水线，分支预测错误惩罚增加一个周期，同时寄存器开销和时钟分布功耗上升。深层流水线假设理想化（无冒险、完美预测），但真实程序中存在不可消除的控制依赖和数据依赖。当级数超过某一点后，IPC 收益为负，功耗成为首要瓶颈（功耗墙）。
  warning_signs:
    - "分支预测准确率低于 95% 但流水线深度超过 20 级"
    - "SPEC 分数提升停滞但芯片 TDP 显著上升"
    - "实际应用 CPI 高于理论模型的 2 倍以上"
  bound_to:
    - "定量性能分析框架"
    - "并发与容错设计原则"
  tags: [pipelining, branch-prediction, power-wall, ipc, amdahl]

- id: ce03
  title: Amdahl 定律不适用
  type: counter-example
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Fallacies and Pitfalls"
  source_quote: |
    "Amdahl's Law doesn't apply to parallel computers."
  failure_mode: |
    设计者忽略串行部分的比例，盲目增加核心数或处理器数量，导致加速比远低于预期，投资回报率极差。
  mechanism: |
    Amdahl 定律指出加速比上限受限于不可并行化的串行比例。即使并行部分无限扩展，总加速比也受 1/(1-p) 的硬边界约束。在并行计算机中，通信、同步和负载不均衡会进一步增加"有效串行比例"，使实际加速比更低。
  warning_signs:
    - "核心数翻倍但应用吞吐量提升不到 30%"
    - "存在无法并行化的初始化或归约阶段"
    - "并行扩展曲线在 8-16 核后明显平坦化"
  bound_to:
    - "定量性能分析框架"
    - "并发与容错设计原则"
  tags: [amdahl, parallel-computing, scalability, speedup, bottleneck]

- id: ce04
  title: 单纯提高时钟频率忽视功耗墙
  type: counter-example
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Fallacies and Pitfalls"
  source_quote: |
    "You can improve performance without affecting energy efficiency."
  failure_mode: |
    通过提高电压和频率榨取性能，导致动态功耗按立方增长，芯片散热不可持续，最终触发热节流（thermal throttling），实际性能反而下降。
  mechanism: |
    动态功耗公式 P = C * V^2 * f。提高频率通常需要同比提高电压，因此功耗与频率近似呈立方关系。当功耗密度超过散热能力（约 100-150 W/cm^2），芯片温度上升，漏电流增加，必须降频运行。这是 2000 年代中期单核频率竞赛终结的根本原因。
  warning_signs:
    - "TDP 接近或超过散热系统极限"
    - "持续高负载下出现热节流导致的频率波动"
    - "性能/Watt 指标随频率提升而恶化"
  bound_to:
    - "定量性能分析框架"
    - "局部性驱动的优化决策"
  tags: [power-wall, thermal-throttling, energy-efficiency, dennard-scaling]

---

## 计算机网络

- id: ce05
  title: 隐藏终端导致 CSMA 失效
  type: counter-example
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "Wireless and Mobile Networks"
  source_quote: |
    "The hidden terminal problem: two nodes may not be able to detect each other's transmissions, leading to collisions at the receiver."
  failure_mode: |
    在无线网络中，发送方因无法感知彼此的信号而同时传输，导致接收端碰撞，CSMA/CD 或基本 CSMA 的冲突避免机制彻底失效。
  mechanism: |
    CSMA 的有效性建立在"发送前能感知介质是否忙"的假设上。在无线环境中，信号衰减和障碍物导致两个发送方互为"隐藏终端"——它们各自能到达接收方，但彼此无法直接通信。因此载波侦听显示"空闲"，但接收端实际发生碰撞。这直接催生了 MACA/MACAW 和 RTS/CTS 握手机制。
  warning_signs:
    - "无线链路吞吐量远低于理论带宽，且随节点数增加急剧下降"
    - "碰撞率与节点密度正相关，但与单个节点的感知负载无关"
    - "增加发送功率反而恶化碰撞（暴露终端问题）"
  bound_to:
    - "并发与容错设计原则"
    - "层次化系统分析方法"
  tags: [wireless, csma, hidden-terminal, collision, rts-cts]

- id: ce06
  title: 增加带宽就能解决所有延迟问题
  type: counter-example
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "Computer Networks and the Internet"
  source_quote: |
    "Bandwidth and latency are not the same; increasing bandwidth does not reduce propagation delay."
  failure_mode: |
    工程师将用户体验差（如网页加载慢、视频会议卡顿）简单归因于带宽不足，升级链路后延迟问题依然存在。
  mechanism: |
    端到端延迟由传播延迟、传输延迟、处理延迟和排队延迟组成。带宽仅影响传输延迟（数据量/带宽），而对传播延迟（距离/光速）无能为力。对于交互式应用（如在线游戏、VoIP），往返传播延迟（RTT）是主导因素；对于小对象请求，TCP 慢启动和握手延迟才是瓶颈。带宽升级对这类问题边际效益极低。
  warning_signs:
    - "带宽利用率长期低于 30% 但用户体验仍差"
    - "延迟敏感型应用（VoIP/游戏）性能未随带宽升级改善"
    - "RTT 占请求总时间的 80% 以上"
  bound_to:
    - "定量性能分析框架"
    - "层次化系统分析方法"
  tags: [bandwidth, latency, rtt, propagation-delay, network-performance]

- id: ce07
  title: 忽略传输层协议与应用层语义的不匹配
  type: counter-example
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "Transport Layer"
  source_quote: |
    "TCP provides a reliable byte-stream abstraction, but applications often need message boundaries or timing semantics that TCP does not preserve."
  failure_mode: |
    应用开发者假设 TCP 按发送边界交付数据，导致消息粘包、解析错误；或在需要实时性的场景误用 TCP，造成不必要的延迟。
  mechanism: |
    TCP 是面向字节流的可靠传输协议，不保留应用层消息边界，也不保证发送端 write() 的时序与接收端 read() 的时序一一对应。应用层必须自行实现帧定界（如长度前缀、分隔符）。此外，TCP 的拥塞控制、重传和按序交付机制会引入不可预测的延迟抖动，对实时音视频等需要低延迟而非绝对可靠的场景反而是负担。
  warning_signs:
    - "接收端解析到半个消息或粘在一起的多条消息"
    - "实时流传输中出现因 TCP 重传导致的秒级卡顿"
    - "应用层未实现消息边界协议直接读取 TCP 流"
  bound_to:
    - "抽象边界设计思维"
    - "层次化系统分析方法"
  tags: [tcp, application-layer, message-framing, real-time, semantic-mismatch]

---

## 数据密集型应用 / 分布式系统

- id: ce08
  title: 忽略网络分区（假设网络可靠）
  type: counter-example
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Reliable, Scalable, and Maintainable Applications"
  source_quote: |
    "Networks are not reliable. You must design for network partitions and partial failures."
  failure_mode: |
    系统设计者假设节点间通信总是成功，未处理超时、丢包和分区，导致在真实网络故障时数据不一致、服务雪崩或脑裂。
  mechanism: |
    分布式系统的根本挑战之一是异步网络：消息可能丢失、延迟或乱序。CAP 定理指出，在分区发生时，系统必须在一致性（C）和可用性（A）之间做出选择。若设计者未显式处理分区，系统往往在压力下以不可预测的方式行为——例如，双主写入导致冲突数据、超时重试引发级联故障、缺乏降级策略导致全面不可用。
  warning_signs:
    - "系统未定义网络分区时的明确行为策略"
    - "跨节点 RPC 调用没有超时和熔断机制"
    - "分布式事务在部分节点失联时挂起或死锁"
  bound_to:
    - "并发与容错设计原则"
    - "数据系统可靠性评估"
  tags: [network-partition, cap-theorem, distributed-systems, fault-tolerance, timeout]

- id: ce09
  title: 过早优化一致性而牺牲可用性
  type: counter-example
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Consistency and Consensus"
  source_quote: |
    "Strong consistency is expensive. Not every application needs linearizability."
  failure_mode: |
    在不需要强一致性的业务场景（如社交点赞、评论数、商品库存预估）强制使用线性一致性或分布式事务，导致系统延迟高、可用性差、扩展困难。
  mechanism: |
    线性一致性要求所有操作看起来按全局顺序原子执行，通常需要共识协议（如 Paxos/Raft）或全局锁，其延迟下限由网络 RTT 决定，且分区时不可用。许多业务场景天然可接受最终一致性（eventual consistency）或因果一致性。过早选择强一致性是对 CAP 权衡的误用，将一致性成本强加于不需要它的应用。
  warning_signs:
    - "读操作延迟明显高于业务需求，且与跨地域 RTT 强相关"
    - "系统在轻微网络抖动时大量读写失败"
    - "业务逻辑本身可接受秒级甚至分钟级不一致"
  bound_to:
    - "数据系统可靠性评估"
    - "并发与容错设计原则"
  tags: [consistency, availability, cap-theorem, linearizability, premature-optimization]

- id: ce10
  title: 将分布式事务当作本地事务处理
  type: counter-example
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Distributed Transactions"
  source_quote: |
    "Distributed transactions are fundamentally different from single-node transactions. They are slow, brittle, and should be avoided if possible."
  failure_mode: |
    开发者将跨服务、跨数据库的操作套用本地 ACID 事务的思维，导致性能瓶颈、死锁频发、故障恢复复杂，系统可用性严重受损。
  mechanism: |
    本地事务依赖共享内存和快速锁机制，可在微秒级完成提交。分布式事务需要两阶段提交（2PC）或三阶段提交（3PC），协调者与参与者之间的多次网络往返使延迟增加数个数量级。更关键的是，2PC 的协调者是单点故障：若协调者在准备阶段后崩溃，参与者必须阻塞等待，直到协调者恢复。这与"高可用"设计目标直接冲突。
  warning_signs:
    - "跨服务调用被包装在全局事务中"
    - "事务提交延迟达到数百毫秒甚至秒级"
    - "协调者故障导致参与者长时间持有锁"
  bound_to:
    - "数据系统可靠性评估"
    - "并发与容错设计原则"
  tags: [distributed-transactions, 2pc, acid, sagas, coupling]

- id: ce11
  title: 信任单一数据源而不验证
  type: counter-example
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Batch Processing"
  source_quote: |
    "Garbage in, garbage out. Data quality issues are often more damaging than software bugs."
  failure_mode: |
    系统直接消费上游数据而不做模式验证、完整性检查或异常检测，导致错误数据在 pipeline 中扩散，最终输出不可靠的决策依据。
  mechanism: |
    数据密集型系统的可靠性不仅取决于代码正确性，还取决于输入数据的质量。上游系统可能因升级、配置错误或临时故障产生异常数据。若下游缺乏显式的数据契约（schema enforcement）、校验和（checksum）或监控告警，错误数据会静默传播，直到在业务端造成可见损失。此时根因分析极其困难，因为错误已被多层转换掩盖。
  warning_signs:
    - "数据 pipeline 中没有 schema 验证或死信队列"
    - "下游指标突变但上游无告警"
    - "空值、重复记录或异常分布未被拦截"
  bound_to:
    - "数据系统可靠性评估"
    - "抽象边界设计思维"
  tags: [data-quality, schema-validation, single-source-of-truth, pipeline, monitoring]

---

## 操作系统

- id: ce12
  title: 错误使用信号量导致死锁
  type: counter-example
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "Concurrency"
  source_quote: |
    "Semaphores are powerful but easy to misuse. Incorrect ordering of P() and V() operations can lead to deadlock."
  failure_mode: |
    多个线程以不同顺序获取多个信号量，形成循环等待，所有线程永久阻塞，系统挂起。
  mechanism: |
    死锁的四个必要条件之一是"循环等待"。信号量作为低级同步原语，不提供死锁预防机制。若线程 A 先获取 sem1 再请求 sem2，而线程 B 先获取 sem2 再请求 sem1，当两者各持有一个信号量并请求另一个时，循环等待形成。由于信号量没有超时或优先级继承，死锁将持续到外部干预。这凸显了低级原语在复杂并发场景中的脆弱性。
  warning_signs:
    - "多个线程/进程在持有锁的同时请求其他锁"
    - "锁的获取顺序在不同代码路径中不一致"
    - "系统间歇性挂起且 CPU 利用率骤降"
  bound_to:
    - "并发与容错设计原则"
    - "抽象边界设计思维"
  tags: [semaphore, deadlock, concurrency, synchronization, circular-wait]

- id: ce13
  title: 忽略崩溃一致性（文件系统元数据损坏）
  type: counter-example
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "Persistence"
  source_quote: |
    "A file system crash can leave the on-disk state inconsistent if updates are not carefully ordered."
  failure_mode: |
    系统崩溃或断电后，文件系统元数据（inode、目录项、位图）处于不一致状态，导致文件丢失、目录损坏或无法挂载。
  mechanism: |
    文件系统操作通常涉及多个磁盘块的更新（如数据块、inode、父目录、空闲块位图）。若这些写操作未按原子顺序刷盘，崩溃可能发生在部分块已写、部分块未写的中间状态。例如，inode 已记录新数据块但位图未标记为已用，导致该块被重复分配。现代文件系统通过 journaling（日志）、copy-on-write 或 soft updates 解决此问题，但应用程序若直接绕过文件系统或使用错误的 fsync 策略，仍可能丢失数据。
  warning_signs:
    - "关键写操作后未调用 fsync/msync"
    - "文件系统在不正常关机后出现 fsck 错误"
    - "数据库或应用日志与数据文件不同步"
  bound_to:
    - "并发与容错设计原则"
    - "数据系统可靠性评估"
  tags: [crash-consistency, filesystem, journaling, fsync, durability]

- id: ce14
  title: 认为更多线程总是更好
  type: counter-example
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "Concurrency"
  source_quote: |
    "Adding more threads does not always improve performance. Beyond a certain point, context switching and cache thrashing dominate."
  failure_mode: |
    为 CPU 密集型任务创建过多线程，导致上下文切换开销、缓存失效和锁竞争激增，实际吞吐量反而下降。
  mechanism: |
    线程是操作系统调度的基本单位。当线程数超过物理核心数时，时间片轮转引发频繁的上下文切换，保存/恢复寄存器和切换地址空间的开销消耗 CPU 周期。更隐蔽的是缓存局部性破坏：不同线程的工作集互相驱逐，导致 L1/L2/L3 缓存命中率骤降。此外，共享数据的锁竞争使线程大量时间花在自旋或阻塞上，形成"并发反扩展"（negative scaling）。
  warning_signs:
    - "线程数超过 CPU 核心数 2-3 倍但 CPU 利用率未饱和"
    - "系统时间 (sys time) 占比随线程数增加而上升"
    - "缓存未命中率与线程数正相关"
  bound_to:
    - "并发与容错设计原则"
    - "定量性能分析框架"
  tags: [threading, context-switch, cache-thrashing, negative-scaling, concurrency]

- id: ce15
  title: 虚拟内存过度配置导致抖动
  type: counter-example
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "Virtual Memory"
  source_quote: |
    "Thrashing occurs when a system spends more time paging than executing useful work."
  failure_mode: |
    同时运行的工作集总和远超物理内存，导致页面频繁换入换出，CPU 利用率暴跌，系统响应极度迟缓。
  mechanism: |
    虚拟内存通过页表将逻辑地址映射到物理页框。当活跃页面集合（working set）大于可用物理内存时，每次内存访问都可能触发缺页中断（page fault），操作系统必须从磁盘（swap）调入页面。由于磁盘 I/O 比内存访问慢 5 个数量级，进程大部分时间处于阻塞等待状态。更糟的是，多个进程竞争有限的页框，形成恶性循环：进程因缺页被调度出去，换入的进程也很快缺页，系统吞吐量趋近于零。
  warning_signs:
    - "磁盘 I/O 持续高负载但应用吞吐量极低"
    - "CPU 利用率低但系统负载 (load average) 极高"
    - "大量 major page faults 且无明显内存泄漏"
  bound_to:
    - "局部性驱动的优化决策"
    - "定量性能分析框架"
  tags: [virtual-memory, thrashing, paging, working-set, memory-pressure]

---

## 数据库系统

- id: ce16
  title: 在没有索引的情况下进行全表扫描
  type: counter-example
  source_books: ["Readings in Database Systems"]
  source_chapter: "Query Processing"
  source_quote: |
    "A full table scan is often the most expensive operation in a database system."
  failure_mode: |
    随着数据量增长，查询响应时间从毫秒级恶化为秒级甚至分钟级，数据库 CPU 和 I/O 被低效查询耗尽，影响所有并发用户。
  mechanism: |
    全表扫描需要读取表中每一行并应用谓词过滤。其时间复杂度为 O(N)，与表大小线性相关。当表达到百万甚至十亿行时，即使简单的点查询也需要遍历海量无关数据。索引（如 B+ 树、哈希索引）将点查询复杂度降至 O(log N) 或 O(1)，并通过聚簇索引减少 I/O 次数。缺乏索引不仅浪费磁盘带宽，还会将热数据从缓存中驱逐，影响其他查询的性能。
  warning_signs:
    - "EXPLAIN 显示 Seq Scan 且返回行数远小于表总行数"
    - "简单 WHERE 条件查询耗时随数据量线性增长"
    - "查询计划中的 rows 估计值与实际值偏差巨大"
  bound_to:
    - "局部性驱动的优化决策"
    - "数据系统可靠性评估"
  tags: [indexing, full-table-scan, query-optimization, b-tree, io-bottleneck]

- id: ce17
  title: 忽略查询优化器的提示
  type: counter-example
  source_books: ["Readings in Database Systems"]
  source_chapter: "Query Optimization"
  source_quote: |
    "The optimizer's cost model is based on statistics. Outdated or missing statistics lead to disastrous plans."
  failure_mode: |
    开发者或 DBA 忽视统计信息更新和优化器提示，导致优化器选择嵌套循环连接处理大表、错误估计基数，生成执行时间高出数个数量级的查询计划。
  mechanism: |
    关系数据库的查询优化器依赖表和索引的统计信息（行数、直方图、唯一值数量）来估算不同执行计划的成本。当统计信息过时（如大量数据加载后未 ANALYZE）或缺失时，优化器的成本模型与现实脱节。例如，它可能认为小表与大表连接时嵌套循环最优，但实际需要哈希连接；或者选择错误的索引，导致随机 I/O 而非顺序扫描。强制使用提示（hint）而不理解其适用条件也会在未来数据分布变化时造成计划退化。
  warning_signs:
    - "ANALYZE 命令长时间未执行"
    - "同一查询在不同时间性能差异巨大"
    - "执行计划中的成本估计与实际执行时间相差 10 倍以上"
  bound_to:
    - "定量性能分析框架"
    - "数据系统可靠性评估"
  tags: [query-optimizer, statistics, cost-model, execution-plan, cardinality-estimation]

- id: ce18
  title: 在 OLTP 系统上运行复杂分析查询
  type: counter-example
  source_books: ["Readings in Database Systems"]
  source_chapter: "Data Warehousing"
  source_quote: |
    "OLTP and OLAP workloads have fundamentally different access patterns and optimization goals."
  failure_mode: |
    在面向短事务的 OLTP 数据库上执行长时间运行的分析查询（如全表聚合、多表连接），阻塞写操作、耗尽连接池，导致在线业务超时或崩溃。
  mechanism: |
    OLTP 工作负载特点是高并发、短事务、随机 I/O、行级访问；OLAP 则是低并发、长事务、顺序 I/O、大量聚合。分析查询通常需要全表扫描和大量内存，与 OLTP 的索引优化和锁粒度设计相冲突。在共享环境中运行 OLAP 查询会导致：锁等待延长、缓冲区污染（将 OLTP 热数据逐出缓存）、连接池耗尽。这是数据仓库与 OLTP 系统分离的根本驱动力。
  warning_signs:
    - "分析查询执行期间 OLTP 事务延迟激增"
    - "数据库连接池被长查询占满"
    - "锁等待时间成为性能瓶颈的首要指标"
  bound_to:
    - "数据系统可靠性评估"
    - "定量性能分析框架"
  tags: [oltp, olap, workload-isolation, data-warehouse, resource-contention]

- id: ce19
  title: 低估并发控制的开销
  type: counter-example
  source_books: ["Readings in Database Systems"]
  source_chapter: "Transaction Processing"
  source_quote: |
    "Concurrency control is not free. Two-phase locking, in particular, can serialize transactions and limit throughput."
  failure_mode: |
    设计者在高并发场景下使用过于保守的隔离级别（如 Serializable）或粗粒度锁，导致事务串行化、死锁频发，系统吞吐量远低于硬件能力。
  mechanism: |
    事务隔离级别是正确性与性能之间的权衡。Serializable 通过两阶段锁（2PL）或串行化快照隔离（SSI）防止所有异常，但会引入大量锁冲突和回滚。对于许多应用，Read Committed 或 Snapshot Isolation 已足够。此外，锁粒度（行锁 vs 表锁）直接影响并发度。低估并发控制开销会导致：锁升级阻塞整个表、热点行竞争形成串行瓶颈、死锁检测和回滚消耗大量 CPU。这要求设计者理解 workload 特性并选择合适的隔离策略。
  warning_signs:
    - "高并发下吞吐量不随核心数增加而提升"
    - "死锁率或锁等待时间占事务总时间比例过高"
    - "业务逻辑实际可接受较低隔离级别但使用了 Serializable"
  bound_to:
    - "并发与容错设计原则"
    - "定量性能分析框架"
  tags: [concurrency-control, isolation-level, 2pl, deadlock, throughput]

---

## SICP / 程序设计

- id: ce20
  title: 使用赋值破坏引用透明性
  type: counter-example
  source_books: ["Structure and Interpretation of Computer Programs"]
  source_chapter: "Assignment and Local State"
  source_quote: |
    "Assignment introduces time into our computational model, destroying the possibility of referential transparency."
  failure_mode: |
    在原本可用纯函数表达的逻辑中引入 set! 或可变状态，导致同一表达式在不同上下文中求值结果不同，调试困难，推理复杂化。
  mechanism: |
    引用透明性（referential transparency）意味着表达式可被其值安全替换而不改变程序语义。赋值操作打破了这一性质，因为它使表达式的值依赖于求值历史（环境状态）。一旦引入赋值，等价替换不再成立，数学归纳法和等式推理失效，程序的正确性证明需要从操作语义（状态机）角度分析。状态还引入了时序依赖和竞态条件，使并发和错误恢复更加困难。
  warning_signs:
    - "同一函数在相同输入下产生不同输出"
    - "调试时需要追踪变量的完整修改历史"
    - "纯函数式重构后因去除赋值而行为改变"
  bound_to:
    - "抽象边界设计思维"
    - "形式化证明与验证思维"
  tags: [referential-transparency, assignment, mutable-state, reasoning, sicp]

- id: ce21
  title: 过早引入状态导致系统复杂化
  type: counter-example
  source_books: ["Structure and Interpretation of Computer Programs"]
  source_chapter: "Modularity, Objects, and State"
  source_quote: |
    "State is a powerful tool for modeling the world, but it is also a source of complexity. We should delay its introduction as long as possible."
  failure_mode: |
    设计者在问题域尚未明确时就引入对象和可变状态，导致模块间耦合增加、测试困难、并发问题频发，系统难以演化。
  mechanism: |
    状态将模块从纯输入输出关系转变为随时间演化的实体，模块接口必须暴露或隐藏其内部历史。这增加了认知负担：调用者不仅要理解输入参数，还要理解对象的当前状态。过早引入状态会固化尚未稳定的设计假设，使重构成本高昂。SICP 提倡先使用过程抽象和数据抽象构建无状态模型，仅在真实需要建模时间演变（如银行账户余额、物理系统）时才引入状态。这一原则与函数式编程和不可变数据结构的现代实践一致。
  warning_signs:
    - "大量对象方法依赖于内部状态的隐式前置条件"
    - "单元测试需要复杂的 setup 和 teardown 来构造状态"
    - "状态变更在模块间产生不可预期的副作用链"
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
  tags: [state, complexity, modularity, premature-state, coupling]

- id: ce22
  title: 用复杂语言特性替代清晰抽象
  type: counter-example
  source_books: ["Structure and Interpretation of Computer Programs"]
  source_chapter: "Metalinguistic Abstraction"
  source_quote: |
    "Programs must be written for people to read, and only incidentally for machines to execute."
  failure_mode: |
    开发者滥用宏、反射、元编程或晦涩语法糖来"炫技"，代码可读性和可维护性急剧下降，团队成员难以理解和修改。
  mechanism: |
    编程语言的高级特性是构建抽象的工具，而非目的本身。当这些特性被用于绕过清晰的设计，而非表达清晰的设计时，它们会引入隐式依赖、非局部控制流和心智模型负担。SICP 强调元语言抽象的力量在于创建适合问题域的 DSL 和清晰接口，而非在底层语言中玩弄技巧。复杂特性还会增加编译/解释的不确定性，使错误定位和行为预测更加困难。
  warning_signs:
    - "大量代码使用团队多数人无法理解的语言特性"
    - "调试时需要深入语言运行时或宏展开细节"
    - "简单需求变更需要修改多处不相关的晦涩代码"
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
  tags: [abstraction, language-features, readability, maintainability, metalinguistic]

---

## 计算机科学中的数学

- id: ce23
  title: 在没有归纳基础的情况下使用归纳法
  type: counter-example
  source_books: ["Mathematics for Computer Science"]
  source_chapter: "Induction"
  source_quote: |
    "A proof by induction requires both a base case and an inductive step. Omitting the base case is a common and fatal error."
  failure_mode: |
    证明者完成了归纳步骤但忽略了基础情况（如 n=0 或 n=1），导致"证明"了一个实际上不成立的命题。
  mechanism: |
    数学归纳法的有效性依赖于良序原理：若 P(0) 成立且 P(n) => P(n+1)，则 P 对所有自然数成立。缺少基础情况时，归纳步骤只是建立了一个蕴含链，但链的起点不存在。例如，可以"证明"所有马都是同一种颜色——归纳步骤在 n=1 到 n=2 的过渡中失效，但即使归纳步骤形式上成立，没有验证 n=1 的基础情况，整个证明也是无效的。这在算法正确性证明中尤为危险，因为边界条件往往是 bug 的藏身之处。
  warning_signs:
    - "证明中直接假设 P(n) 并推导 P(n+1)，未验证最小 n"
    - "算法在空输入或单元素输入时行为异常"
    - "归纳假设的适用范围与命题定义域不完全匹配"
  bound_to:
    - "形式化证明与验证思维"
    - "抽象边界设计思维"
  tags: [induction, base-case, proof, correctness, boundary-condition]

- id: ce24
  title: 将概率直觉等同于形式化证明
  type: counter-example
  source_books: ["Mathematics for Computer Science"]
  source_chapter: "Random Processes"
  source_quote: |
    "Intuition about probability is notoriously unreliable. Formal definitions and careful reasoning are essential."
  failure_mode: |
    设计者基于"感觉上概率很低"做出安全假设，未进行形式化分析，导致系统在真实攻击或极端负载下以高于预期的概率失败。
  mechanism: |
    人类直觉在处理条件概率、联合概率、大数定律和尾部事件时存在系统性偏差（如生日悖论、赌徒谬误、幸存者偏差）。在系统设计中，"这个事件几乎不可能发生"的直觉往往低估了在大量请求下的累积概率。例如，假设单个磁盘年故障率为 2% 看似很低，但在 1000 块磁盘的集群中，几乎可以肯定年内会发生故障。形式化概率分析要求明确定义样本空间、事件和概率测度，并通过联合界、切尔诺夫界等工具给出严格的上下界。
  warning_signs:
    - "可靠性估计基于直觉而非显式概率计算"
    - "未考虑独立事件在规模放大后的累积效应"
    - "对尾部风险（tail risk）缺乏定量分析"
  bound_to:
    - "形式化证明与验证思维"
    - "定量性能分析框架"
  tags: [probability, intuition, formal-proof, tail-risk, reliability]

- id: ce25
  title: 忽略边界条件
  type: counter-example
  source_books: ["Mathematics for Computer Science"]
  source_chapter: "Proofs and Induction"
  source_quote: |
    "Boundary cases are where most errors hide. A proof or algorithm that works 'in the middle' often fails at the edges."
  failure_mode: |
    算法或证明在典型输入下表现正确，但在空集、单元素、最大值/最小值等边界条件下崩溃或产生错误结果。
  mechanism: |
    边界条件往往触发与常规情况不同的控制流：循环零次执行、递归直接返回、除零、整数溢出、空指针解引用。设计者在构造证明或算法时，通常从"一般情况"出发，心智模型聚焦于 n 较大时的行为模式，从而忽视 n=0, n=1 或极端值时的异常。数学归纳法要求基础情况正是为了强制检查这些边界。在软件工程中，fuzzing 和 property-based testing 的核心价值也在于自动探索边界条件。
  warning_signs:
    - "单元测试未覆盖空输入、单元素输入或最大值输入"
    - "证明中基础情况被一笔带过或完全省略"
    - "算法在特定阈值（如数组长度 0 或 1）处行为异常"
  bound_to:
    - "形式化证明与验证思维"
    - "抽象边界设计思维"
  tags: [boundary-condition, edge-case, correctness, induction, testing]

---

## 高级算法与数据结构

- id: ce26
  title: 在需要精确答案时使用概率数据结构
  type: counter-example
  source_books: ["Advanced Algorithms and Data Structures"]
  source_chapter: "Probabilistic Data Structures"
  source_quote: |
    "Probabilistic data structures trade exactness for space and time efficiency. They are dangerous when exact answers are required."
  failure_mode: |
    在需要精确计数的场景（如金融交易、库存管理、安全审计）使用 HyperLogLog 或 Count-Min Sketch，导致不可接受的统计误差，造成资金损失或合规风险。
  mechanism: |
    概率数据结构通过哈希冲突和随机化来换取极低的内存占用和 O(1) 更新时间，但其输出本质上是带有置信区间的估计值。例如，HyperLogLog 的基数估计存在标准误差（通常为 1-2%），Count-Min Sketch 在查询频率时可能高估（因哈希碰撞）。这些误差在"近似即可"的场景（如 UV 统计、热点检测）是可接受的，但在需要精确性的领域，误差会被放大为系统性风险。设计者必须明确区分"可接受的近似"与"不可妥协的精确"。
  warning_signs:
    - "业务逻辑要求 100% 准确但使用了概率性计数器"
    - "审计或合规场景中出现无法解释的数据偏差"
    - "误差范围未向业务方明确沟通"
  bound_to:
    - "定量性能分析框架"
    - "数据系统可靠性评估"
  tags: [probabilistic-data-structures, hyperloglog, count-min-sketch, approximation, exactness]

- id: ce27
  title: 忽略 Bloom filter 的假阳性率
  type: counter-example
  source_books: ["Advanced Algorithms and Data Structures"]
  source_chapter: "Probabilistic Data Structures"
  source_quote: |
    "Bloom filters can return false positives. The false positive rate must be explicitly modeled and bounded."
  failure_mode: |
    将 Bloom filter 用于存在性检查的前置过滤，但未考虑假阳性率，导致大量不存在的数据被误判为存在，引发不必要的下游计算或缓存未命中。
  mechanism: |
    Bloom filter 通过多个哈希函数将元素映射到位数组，其空间效率来自于允许可控的假阳性（false positives）——即元素实际不存在但查询返回可能存在。假阳性率由位数组大小 m、元素数量 n 和哈希函数数量 k 决定，理论值为 (1 - e^(-kn/m))^k。若设计者未根据预期数据量调整 m 和 k，或数据量增长超出设计容量，假阳性率会迅速上升。在数据库查询优化、缓存穿透防护等场景中，未建模的假阳性会直接降低系统效率。
  warning_signs:
    - "Bloom filter 查询返回 '可能存在' 后下游命中率极低"
    - "未根据数据增长动态调整 filter 大小"
    - "假阳性率未作为监控指标被追踪"
  bound_to:
    - "定量性能分析框架"
    - "数据系统可靠性评估"
  tags: [bloom-filter, false-positive, probabilistic, cache, query-optimization]

- id: ce28
  title: 不考虑缓存局部性而只追求渐近最优
  type: counter-example
  source_books: ["Advanced Algorithms and Data Structures"]
  source_chapter: "Cache-Oblivious Algorithms"
  source_quote: |
    "Asymptotic optimality does not guarantee practical performance. Memory hierarchy and cache behavior often dominate."
  failure_mode: |
    算法在理论上具有最优时间复杂度，但由于缺乏空间局部性，缓存未命中频繁，实际运行时间被内存访问延迟主导，远低于理论预期。
  mechanism: |
    现代计算机的内存层次结构（寄存器-L1-L2-L3-主存）意味着数据访问成本差异可达数个数量级。传统渐近分析基于均匀内存访问（RAM）模型，忽略了缓存行、预取和 TLB 的影响。例如，链表遍历的 O(n) 理论上与数组扫描相同，但由于每个节点可能触发独立的缓存未命中，实际性能可能慢 10-100 倍。类似地，某些分治算法若未针对缓存行大小设计分块策略，会导致大量随机内存访问。 cache-oblivious 和 cache-aware 算法正是为了弥合这一鸿沟。
  warning_signs:
    - "理论复杂度低的算法实测性能不如'更简单'的实现"
    - "性能分析显示大量 LLC (Last Level Cache) 未命中"
    - "数据访问模式呈现高度随机性而非顺序性"
  bound_to:
    - "局部性驱动的优化决策"
    - "定量性能分析框架"
  tags: [cache-locality, asymptotic-complexity, memory-hierarchy, performance-gap, practical-efficiency]

---

## 跨书通用反例

- id: ce29
  title: 抽象边界与实现细节耦合
  type: counter-example
  source_books: ["Structure and Interpretation of Computer Programs", "Designing Data-Intensive Applications"]
  source_chapter: "Data Abstraction / Evolvability"
  source_quote: |
    "The boundary between interface and implementation is the most important design decision in any system."
  failure_mode: |
    客户端直接依赖内部数据结构或实现细节，导致任何内部重构都会引发级联修改，系统演化能力被严重削弱。
  mechanism: |
    抽象的核心价值在于隔离变化。当接口未能有效隐藏实现细节时，模块间的契约从"行为契约"退化为"结构契约"。这意味着实现者失去了优化内部表示的自由，任何性能改进或架构调整都可能破坏下游依赖。SICP 中的数据抽象和 DDIA 中的 schema 演进都强调：稳定的接口是系统长期可维护性的前提。微服务中的 API 版本控制、数据库中的视图和存储过程、面向对象中的封装，都是这一原则的具体实践。
  warning_signs:
    - "客户端代码直接访问模块内部数据结构"
    - "API 响应格式与内部数据库 schema 一一对应"
    - "内部重构需要修改大量外部调用方"
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
  tags: [abstraction, encapsulation, coupling, evolvability, interface-design]

- id: ce30
  title: 在没有测量的情况下进行优化
  type: counter-example
  source_books: ["Computer Architecture: A Quantitative Approach", "Designing Data-Intensive Applications"]
  source_chapter: "Performance / Quantitative Approach"
  source_quote: |
    "Premature optimization is the root of all evil. You cannot improve what you do not measure."
  failure_mode: |
    工程师凭直觉对自认为的瓶颈进行优化，投入大量精力重构代码或引入复杂技术，但实际瓶颈位于别处，整体性能几乎没有改善。
  mechanism: |
    系统性能由多个子系统和资源共同决定，瓶颈可能出现在出乎意料的地方（如网络延迟、锁竞争、垃圾回收、数据库连接池）。没有基于 profiling 和基准测试的量化分析，优化决策本质上是随机的。Amdahl 定律进一步指出：对非瓶颈部分的优化无论多么成功，对整体加速比的贡献都极其有限。定量方法要求先建立测量基线，识别瓶颈，评估改进潜力，再实施优化。
  warning_signs:
    - "优化前未进行 profiling 或基准测试"
    - "优化后未验证实际性能提升"
    - "投入大量精力优化的组件在整体延迟中占比不足 5%"
  bound_to:
    - "定量性能分析框架"
    - "局部性驱动的优化决策"
  tags: [premature-optimization, profiling, benchmarking, bottleneck, measurement]

---

## 统计

- 共提取 **30 条** counter-examples
- 覆盖 8 本书中的 7 个主要领域（体系结构、网络、DDIA、OS、数据库、SICP、数学、算法）
- 每条均包含：failure_mode、mechanism、warning_signs、bound_to、tags
- 所有反例均已绑定到 `BOOK_OVERVIEW.md` 中识别的正面 skill 候选
