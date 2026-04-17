# CS Core Thinking — Extracted Frameworks (Phase 1)

> Extracted by framework-extractor from the 8-book meta-framework.
> Source books abbreviated: CAQA = Computer Architecture: A Quantitative Approach; CN = Computer Networking: A Top-Down Approach; DDIA = Designing Data-Intensive Applications; OSTEP = Operating Systems: Three Easy Pieces; RedBook = Readings in Database Systems; SICP = Structure and Interpretation of Computer Programs; MCS = Mathematics for Computer Science; AADS = Advanced Algorithms and Data Structures.

---

- id: f01
  title: Amdahl定律驱动的加速上限评估框架
  type: framework
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Chapter 1.9 — Quantitative Principles of Computer Design"
  source_quote: |
    "Amdahl's Law states that the performance improvement to be gained from using some faster mode of execution is limited by the fraction of the time the faster mode can be used."
  summary: |
    适用场景：评估任何局部优化（硬件加速、算法改进、并行化）对整体系统的真实收益上限。
    推理步骤：
    1) 测量或估计被优化部分在原执行时间中的占比 Fraction_enhanced；
    2) 确定优化后的加速比 Speedup_enhanced；
    3) 代入 Amdahl 公式计算整体加速上限；
    4) 若加速上限低于业务目标，则放弃单点优化，转向架构重构。
    核心假设：被优化部分的加速与未优化部分相互独立；工作负载不变。
  tags: [quantitative-analysis, performance-engineering, speedup-bounds, architecture]

- id: f02
  title: 局部性驱动的存储层次优化决策框架
  type: framework
  source_books: ["Computer Architecture: A Quantitative Approach", "Operating Systems: Three Easy Pieces", "Advanced Algorithms and Data Structures"]
  source_chapter: "CAQA Ch.1.9 / OSTEP Ch.19–22 / AADS Ch.2"
  source_quote: |
    "The most important program property that we regularly exploit is the principle of locality: programs tend to reuse data and instructions they have used recently. Temporal locality states that recently accessed items are likely to be accessed soon. Spatial locality says that items whose addresses are near one another tend to be referenced close together in time."
  summary: |
    适用场景：缓存设计、页表策略、磁盘调度、数据结构布局、预取算法。
    推理步骤：
    1) 分析工作负载的访问模式，识别时间局部性（循环、热点数据）与空间局部性（顺序遍历、数组/矩阵）；
    2) 若时间局部性高，优先增大缓存容量或提高关联度；
    3) 若空间局部性高，优先采用预取、增大块大小（block size）或调整数据布局；
    4) 若两者皆低，则存储层次优化收益有限，需考虑算法重构。
    核心假设：程序访问模式在短期内保持稳定；存储层次的成本-延迟差异显著。
  tags: [locality, memory-hierarchy, cache-design, data-layout, prefetching]

- id: f03
  title: 定量性能分析的三元方程框架（CPU Time = IC × CPI × Cycle Time）
  type: framework
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "Chapter 1.8–1.9 — Measuring Performance & Quantitative Principles"
  source_quote: |
    "CPU time = Instruction count × Clock cycles per instruction × Clock cycle time. As this formula demonstrates, processor performance is dependent upon three characteristics: clock cycle (or rate), clock cycles per instruction, and instruction count."
  summary: |
    适用场景：CPU/微架构级别的性能诊断与优化决策。
    推理步骤：
    1) 将总执行时间拆解为指令数（IC）、每条指令周期数（CPI）、时钟周期时间（T）；
    2) 通过性能计数器或模拟器分别测量三项指标；
    3) 识别瓶颈项：若 IC 过高，优化编译器或算法；若 CPI 过高，优化流水线/缓存/分支预测；若 T 过大，考虑更高频率或更先进工艺；
    4) 评估每项改进对总 CPU time 的边际贡献，避免对非瓶颈项过度投入。
    核心假设：程序行为可被稳定采样；性能计数器准确。
  tags: [quantitative-analysis, cpu-performance, measurement, bottleneck-analysis]

- id: f04
  title: 分层抽象与端到端原则决策框架
  type: framework
  source_books: ["Computer Networking: A Top-Down Approach", "SICP", "Operating Systems: Three Easy Pieces"]
  source_chapter: "CN Preface & Ch.1 / SICP Ch.1 / OSTEP Ch.36"
  source_quote: |
    "Our book broke new ground by treating networking in a top-down manner—that is, by beginning at the application layer and working its way down toward the physical layer." (CN)
    "We control complexity by building abstractions that hide details when appropriate. We control complexity by establishing conventional interfaces that enable us to construct systems by combining standard, well-understood pieces." (SICP)
  summary: |
    适用场景：设计跨层系统（网络协议栈、操作系统、软件架构）时决定功能应放在哪一层实现。
    推理步骤：
    1) 明确需求：功能是否只能由端系统准确判断？若网络核心无法获得完整语义，则遵循端到端原则，将功能放在端系统实现；
    2) 若功能需要全局状态或实时路径信息，则考虑在网络层/中间层实现；
    3) 评估分层实现的代价：每增加一层抽象，带来接口开销但降低耦合度；
    4) 选择使“接口简洁、实现可替换、故障隔离清晰”的抽象边界。
    核心假设：端系统可信；网络核心以转发为主；抽象层的接口稳定。
  tags: [abstraction, layering, end-to-end-principle, system-design, modularity]

- id: f05
  title: TCP 拥塞控制的 AIMD 带宽探测框架
  type: framework
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "Chapter 3.7 — TCP Congestion Control"
  source_quote: |
    "TCP congestion control consists of linear (additive) increase in cwnd of 1 MSS per RTT and then a halving (multiplicative decrease) of cwnd on a triple duplicate-ACK event. For this reason, TCP congestion control is often referred to as an additive-increase, multiplicative-decrease (AIMD) form of congestion control."
  summary: |
    适用场景：设计分布式系统的速率控制、负载调节、资源探测算法。
    推理步骤：
    1) 定义“拥塞信号”：在通用网络中可以是丢包、延迟增长、队列长度；在应用层可以是错误率、P99 延迟、背压信号；
    2) 无负反馈时，采用保守的线性增长探测可用容量（Additive Increase）；
    3) 检测到拥塞信号时，采用乘法级衰减（Multiplicative Decrease）快速释放压力；
    4) 循环 2–3，形成“锯齿形”稳态，使系统收敛到公平且高效的共享带宽。
    核心假设：反馈信号与真实拥塞强相关；多个竞争源共享瓶颈资源。
  tags: [congestion-control, AIMD, rate-adaptation, distributed-systems, networking]

- id: f06
  title: 可靠传输的“确认-重传-超时”三元框架
  type: framework
  source_books: ["Computer Networking: A Top-Down Approach", "Operating Systems: Three Easy Pieces"]
  source_chapter: "CN Ch.3.4–3.5 / OSTEP Ch.36–37"
  source_quote: |
    "TCP provides reliable data transfer by using positive acknowledgments and timers in much the same way that we studied in Section 3.4. TCP acknowledges data that has been received correctly, and it then retransmits segments when segments or their corresponding acknowledgments are thought to be lost or corrupted."
  summary: |
    适用场景：在不可靠介质（网络、消息队列、分布式存储）上构建可靠传输或持久化通道。
    推理步骤：
    1) 为每个发送单元分配序列号，使接收方能检测丢包、重复、乱序；
    2) 接收方返回肯定确认（ACK），ACK 可累积或选择；
    3) 发送方维护超时计时器（Timeout），超时未收到 ACK 则重传；
    4) 使用 RTT 采样动态调整超时阈值（EstimatedRTT + 4·DevRTT），避免过早重传或恢复过慢；
    5) 引入快速重传（Fast Retransmit）：收到 3 个重复 ACK 即视为丢包信号，无需等待超时。
    核心假设：丢包主要由拥塞或链路错误引起；ACK 通道本身也可能丢失但概率较低。
  tags: [reliable-transport, ACK-retransmission, timeout, TCP, distributed-systems]

- id: f07
  title: RSM（Reliable, Scalable, Maintainable）系统设计评估框架
  type: framework
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Chapter 1 — Reliable, Scalable, and Maintainable Applications"
  source_quote: |
    "In this book, we focus on three concerns that are important in most software systems: Reliability — The system should continue to work correctly even in the face of adversity. Scalability — As the system grows, there should be reasonable ways of dealing with that growth. Maintainability — Over time, many different people will work on the system, and they should all be able to work on it productively."
  summary: |
    适用场景：数据密集型应用（后端、数据库、大数据平台）的架构评审与演进决策。
    推理步骤：
    1) 可靠性：识别故障类型（硬件故障、软件错误、人为错误），为每种故障设计容错机制（冗余、优雅降级、混沌测试）；
    2) 可扩展性：用负载参数（请求/秒、读写比、fan-out）描述当前负载，再用性能指标（吞吐量、延迟分位数）评估增长时的行为；
    3) 可维护性：评估系统的可操作性（监控、可观测性）、简单性（抽象边界清晰）、可演化性（接口与实现分离）；
    4) 任何重大架构变更必须同时说明对 R/S/M 三者的影响，避免只优化单一维度。
    核心假设：系统生命周期长；团队会变化；故障不可避免。
  tags: [RSM, reliability, scalability, maintainability, data-intensive-systems]

- id: f08
  title: 负载-性能建模与分位数驱动优化框架
  type: framework
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "Chapter 1 — Describing Load and Performance"
  source_quote: |
    "Load can be described with a few numbers which we call load parameters. The best choice of parameters depends on the architecture of your system." / "High percentiles of response times, also known as tail latencies, are important because they directly affect users' experience of the service."
  summary: |
    适用场景：容量规划、SLA 制定、性能优化、架构选型。
    推理步骤：
    1) 选定与系统架构最相关的负载参数（如请求/秒、fan-out、活跃并发数、缓存命中率）；
    2) 描述性能时，不仅报告平均值，更报告中位数（p50）与尾部延迟（p95/p99/p999）；
    3) 当负载增长时，回答两个问题：a) 资源不变时性能如何变化？b) 要保持性能不变需要增加多少资源？
    4) 若尾部延迟恶化，优先排查同步阻塞点、慢节点、级联依赖；
    5) 对高价值用户（数据量大、付费高）特别关注 p99.9，因为他们往往对应最慢请求。
    核心假设：负载参数可测量；性能分布高度偏斜，平均值具有误导性。
  tags: [load-modeling, tail-latency, performance-metrics, capacity-planning, SLO]

- id: f09
  title: 分布式一致性设计的 CAP/PACELC 决策矩阵
  type: framework
  source_books: ["Designing Data-Intensive Applications", "Readings in Database Systems"]
  source_chapter: "DDIA Ch.7 & Ch.9 / RedBook Ch.6 — Weak Isolation and Distribution"
  source_quote: |
    "There is some similarity between distributed consistency models and the hierarchy of transaction isolation levels we discussed previously. But while there is some overlap, they are mostly independent concerns: transaction isolation is primarily about avoiding race conditions due to concurrently executing transactions, whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults."
  summary: |
    适用场景：设计分布式数据库、缓存、消息系统时选择一致性级别与容错策略。
    推理步骤：
    1) 识别业务对分区容错（P）的容忍度：若网络分区不可接受，可优先保证 CA（但现实中 P 几乎不可避免）；
    2) 在分区发生时，选择 CP（牺牲可用性，保证一致性）或 AP（牺牲强一致性，保证可用性）；
    3) 在无分区时（PACELC 的 EL），选择延迟（Latency）与一致性（Consistency）的权衡：同步复制保证一致性但增加延迟，异步复制降低延迟但引入复制滞后；
    4) 将一致性模型映射到具体实现：线性一致性（Linearizability）→ 单主/共识；顺序一致性 → 分布式日志；最终一致性 → 多主/无主复制；
    5) 对关键操作（锁、选主、 fencing token）必须使用线性一致性或共识。
    核心假设：网络延迟有界但不可预测；节点可能崩溃或 GC 暂停。
  tags: [CAP, PACELC, distributed-consistency, linearizability, fault-tolerance]

- id: f10
  title: 并发控制的悲观-乐观策略选择框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces", "Readings in Database Systems", "Designing Data-Intensive Applications"]
  source_chapter: "OSTEP Ch.28–31 / RedBook Ch.3 / DDIA Ch.7"
  source_quote: |
    "Our first paper on transactions, from Gray et al., introduces two classic ideas: multi-granularity locking and multiple lock modes." (RedBook)
    "Serializable snapshot isolation (SSI) is not linearizable: by design, it makes reads from a consistent snapshot, to avoid lock contention between readers and writers." (DDIA)
  summary: |
    适用场景：数据库事务、并发数据结构、分布式锁的设计与选型。
    推理步骤：
    1) 分析冲突概率：若写冲突频繁、事务竞争激烈，优先采用悲观策略（锁、两阶段锁 2PL），避免大量回滚；
    2) 若读多写少、冲突稀少，优先采用乐观策略（乐观并发控制 OCC、MVCC、Serializable Snapshot Isolation），降低锁开销、提高读吞吐；
    3) 评估事务长度：长事务在悲观锁下易导致死锁与吞吐量骤降，可考虑乐观或混合策略；
    4) 在分布式场景中，若网络延迟高，悲观锁的持有时间会放大，优先采用基于版本向量或冲突检测的乐观策略；
    5) 对必须严格串行化的关键财务操作，使用两阶段锁或实际串行执行。
    核心假设：冲突模式可预测；回滚成本低于锁等待成本时乐观策略占优。
  tags: [concurrency-control, pessimistic-locking, optimistic-concurrency, MVCC, 2PL]

- id: f11
  title: 虚拟化作为资源抽象的统一框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces", "Computer Architecture: A Quantitative Approach"]
  source_chapter: "OSTEP Ch.2 & Ch.4 / CAQA Ch.2.4"
  source_quote: |
    "Virtualizing the CPU... Virtualizing Memory... The three pillars of operating systems: virtualization, concurrency, and persistence." (OSTEP)
    "Virtual machines and virtual memory are two of the most important abstractions in computer systems." (CAQA)
  summary: |
    适用场景：操作系统、云计算、容器平台、硬件虚拟化的设计与资源管理。
    推理步骤：
    1) 识别物理资源的限制与异构性（CPU、内存、磁盘、网络）；
    2) 在资源之上引入虚拟层，将物理资源转化为更通用、更易用的抽象（进程、地址空间、虚拟磁盘、虚拟网络）；
    3) 通过受控的直接执行（Limited Direct Execution）+ 特权级切换/陷入-模拟，保证性能接近原生；
    4) 通过多路复用（时分/空分）让多个用户/应用共享物理资源，同时保持隔离；
    5) 评估虚拟化开销：若上下文切换、页表遍历、设备模拟成为瓶颈，考虑硬件辅助虚拟化（VT-x、IOMMU）或半虚拟化。
    核心假设：用户对抽象接口的调用模式可预测；隔离性优先于极致性能。
  tags: [virtualization, abstraction, resource-management, cloud-computing, isolation]

- id: f12
  title: 调度策略的“工作负载假设-指标-策略”推导框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "OSTEP Ch.7–10 — Scheduling"
  source_quote: |
    "How should we develop a basic framework for thinking about scheduling policies? What are the key assumptions? What metrics are important? What basic approaches have been used in the earliest of computer systems?"
  summary: |
    适用场景：操作系统调度、任务队列、批处理系统、实时系统的策略设计。
    推理步骤：
    1) 明确工作负载假设：作业到达模式、运行时间分布、I/O 密集度、是否可抢占；
    2) 选定优化指标：周转时间（Turnaround Time）、响应时间（Response Time）、公平性（Fairness）、吞吐量（Throughput）、截止时间满足率；
    3) 根据指标选择基础策略：最小化平均周转时间 → SJF/SRTF；最小化响应时间 → RR（时间片轮转）；保证比例公平 → Lottery/Stride/CFS；
    4) 逐步放宽不切实际的假设（如“运行时间已知”），引入多级反馈队列（MLFQ）等自适应机制；
    5) 在多处理器场景下，增加缓存亲和性（Cache Affinity）与负载均衡（Load Balance）作为额外约束，选择单队列（SQMS）或多队列（MQMS）。
    核心假设：调度开销远小于任务执行时间；上下文切换可被硬件支持。
  tags: [scheduling, workload-modeling, turnaround-time, response-time, fairness]

- id: f13
  title: 崩溃恢复与日志结构持久化框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces", "Readings in Database Systems", "Designing Data-Intensive Applications"]
  source_chapter: "OSTEP Ch.42–43 / RedBook Ch.3 (ARIES) / DDIA Ch.7"
  source_quote: |
    "The general way to solve this problem is to use a write-ahead log of some kind to first record what the RAID is about to do before doing it." (OSTEP)
    "ARIES: A Transaction Recovery Method Supporting Fine-Granularity Locking and Partial Rollbacks Using Write-Ahead Logging." (RedBook)
  summary: |
    适用场景：文件系统、数据库、分布式存储的崩溃恢复与持久化设计。
    推理步骤：
    1) 识别更新操作的原子性需求：单个 512B 写可能原子，但跨块更新非原子；
    2) 采用预写日志（Write-Ahead Logging, WAL）：先将对元数据的修改记录到日志并提交（TxE），再异步刷到最终位置（Checkpoint）；
    3) 崩溃后通过重放日志（Redo）恢复已提交事务，通过撤销（Undo）回滚未提交事务；
    4) 评估日志模式：全数据日志（Data Journaling）保证最强一致性但写放大高；元数据日志（Metadata Journaling）平衡性能与一致性；
    5) 若写放大成为瓶颈，考虑日志结构文件系统（LFS）或 Copy-on-Write（ZFS/Btrfs），将随机写转为顺序写。
    核心假设：日志写入顺序可被存储设备保证；崩溃后日志可读。
  tags: [crash-recovery, WAL, journaling, persistence, LFS, COW]

- id: f14
  title: RAID 容量-可靠性-性能三维评估框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "OSTEP Ch.38 — Redundant Arrays of Inexpensive Disks (RAIDs)"
  source_quote: |
    "We will evaluate each RAID design along three axes. The first axis is capacity... The second axis of evaluation is reliability... Finally, the third axis is performance."
  summary: |
    适用场景：存储系统设计、分布式文件系统、数据仓库的冗余与性能权衡。
    推理步骤：
    1) 容量轴：给定 N 块磁盘，计算有效容量（RAID-0 为 N·B，RAID-1 为 N·B/2，RAID-5 为 (N-1)·B）；
    2) 可靠性轴：确定可容忍的磁盘故障数（RAID-0 为 0，RAID-1 为 1 至 N/2，RAID-5 为 1，RAID-6 为 2）；
    3) 性能轴：区分顺序/随机、读/写工作负载，计算稳态带宽（RAID-1 随机读为 N·R，顺序写为 N·S/2；RAID-4 小写受奇偶校验盘瓶颈）；
    4) 根据业务优先级选择 RAID 级别：高读性能+高可靠 → RAID-10；大容量+成本敏感 → RAID-5/6；
    5) 处理一致更新问题（Consistent-Update Problem）：使用 NVRAM 日志或电池保护缓存保证多盘写入的原子性。
    核心假设：磁盘故障独立且可被检测（fail-stop 模型）；工作负载可被分类为顺序或随机。
  tags: [RAID, storage-systems, capacity-reliability-performance, redundancy, disk-arrays]

- id: f15
  title: 查询优化器的成本模型与搜索空间框架
  type: framework
  source_books: ["Readings in Database Systems"]
  source_chapter: "RedBook Ch.3 & Ch.7 — Query Optimization"
  source_quote: |
    "Query optimization is one of the signature components of database technology—the bridge that connects declarative languages to efficient execution." / "No query optimizer is truly producing 'optimal' plans. First, they all use estimation techniques to guess at real plan costs, and it's well known that errors in these estimation techniques can balloon. Second, optimizers use heuristics to limit the search space of plans they choose, since the problem is NP-hard."
  summary: |
    适用场景：数据库查询优化、大数据 SQL 引擎、数据流系统的执行计划生成。
    推理步骤：
    1) 建立统计模型：收集表基数、列分布直方图、索引选择性，用于估算各操作符的代价；
    2) 生成逻辑计划空间：基于关系代数等价变换（选择下推、投影消除、连接重排序）扩展候选计划；
    3) 使用动态规划（System R 风格自底向上）或分支定界（Volcano/Cascades 风格自顶向下）在计划空间中搜索最低代价计划；
    4) 引入启发式剪枝：限制连接顺序搜索深度、优先使用索引、避免笛卡尔积；
    5) 在运行时引入自适应优化（Eddies / Progressive Optimization）：当实际统计与估计严重偏离时，中途中止并重新优化剩余计划。
    核心假设：统计信息相对准确；查询模式稳定；优化时间远小于执行时间。
  tags: [query-optimization, cost-model, plan-space, database-systems, adaptive-query-processing]

- id: f16
  title: 抽象屏障与接口-实现分离的演化框架
  type: framework
  source_books: ["SICP", "Readings in Database Systems", "Designing Data-Intensive Applications"]
  source_chapter: "SICP Ch.1 & Ch.2 / RedBook Ch.1 / DDIA Ch.1 & Ch.4"
  source_quote: |
    "We control complexity by building abstractions that hide details when appropriate. We control complexity by establishing conventional interfaces that enable us to construct systems by combining standard, well-understood pieces in a 'mix and match' way." (SICP)
    "Data independence is the separation of the logical organization of data from its physical storage." (RedBook)
  summary: |
    适用场景：软件架构设计、API 设计、数据库 schema 演进、长期维护系统的接口治理。
    推理步骤：
    1) 定义抽象屏障：将系统划分为多层，每层只暴露必要的操作接口，隐藏内部表示与算法；
    2) 保证接口与实现的正交性：同一接口可对应多种实现（如 B-Tree vs LSM-Tree、行存 vs 列存），实现可独立演进；
    3) 在数据系统中，维护逻辑独立性与物理独立性：修改 schema 不应影响应用查询；修改存储格式不应影响逻辑视图；
    4) 当需求变化时，优先在现有抽象层内调整实现，而非破坏接口契约；若抽象边界失效，则重构屏障而非在层间打洞；
    5) 通过版本化、兼容性测试、渐进式发布，降低接口变更对下游的破坏。
    核心假设：系统生命周期长；需求会变化；团队对实现细节的理解会流失。
  tags: [abstraction-barrier, interface-separation, data-independence, evolvability, software-architecture]

- id: f17
  title: 高阶过程与“程序即数据”的元语言抽象框架
  type: framework
  source_books: ["SICP"]
  source_chapter: "SICP Ch.1.3 & Ch.4 — Higher-Order Procedures and Metalinguistic Abstraction"
  source_quote: |
    "Every powerful language has three mechanisms for accomplishing this: primitive expressions, means of combination, and means of abstraction, by which compound elements can be named and manipulated as units." / "In teaching our material we use a dialect of the programming language Lisp... the metalinguistic power that derives from the simple syntax, the uniform representation of programs as data objects."
  summary: |
    适用场景：DSL 设计、编译器/解释器构建、配置即代码、宏系统、函数式编程。
    推理步骤：
    1) 识别领域中的重复模式，将其抽象为高阶过程（以过程为参数或返回值），消除样板代码；
    2) 当领域语义无法被宿主语言直接表达时，设计领域专用语言（DSL）；
    3) 利用“程序即数据”的同质性（homoiconicity），将 DSL 程序表示为宿主语言的数据结构，从而可用宿主语言的全部工具（解析、转换、求值）处理 DSL；
    4) 构建元循环解释器（Metacircular Evaluator）：用宿主语言实现目标语言的求值-应用循环，验证语义设计；
    5) 在需要性能时，将 DSL 编译为更低级表示（如寄存器机器模型），保留高层抽象的语义不变性。
    核心假设：宿主语言具备足够的数据结构表达能力；DSL 的语义可被形式化定义。
  tags: [higher-order-procedures, metalinguistic-abstraction, DSL, interpreter, homoiconicity]

- id: f18
  title: 归纳证明与不变量原理的形式化验证框架
  type: framework
  source_books: ["Mathematics for Computer Science"]
  source_chapter: "MCS Ch.5 — Induction & Invariant Principle"
  source_quote: |
    "Induction is a powerful method for showing a property is true for all nonnegative integers. It also introduces the Invariant Principle, which is a version of induction specially adapted for reasoning about step-by-step processes."
  summary: |
    适用场景：算法正确性证明、并发协议验证、循环不变量推导、分布式系统安全性分析。
    推理步骤：
    1) 对于递归/迭代结构，定义谓词 P(n) 为待证性质；验证基例 P(0) 成立；
    2) 假设 P(n) 成立，推导 P(n+1) 成立，完成归纳步；
    3) 对于状态机或逐步执行过程，定义不变量（Invariant）：在初始状态成立，且在每一步转移后保持成立；
    4) 将算法终止条件与不变量结合，推出终止时目标性质必然成立；
    5) 在并发/分布式场景中，使用结构归纳或历史归纳（History Induction）证明安全属性（Safety：nothing bad happens）与活性属性（Liveness：something good eventually happens）。
    核心假设：状态转移规则定义良好；初始状态满足前提；步骤有限或良基。
  tags: [induction, invariant-principle, formal-verification, correctness-proof, safety-liveness]

- id: f19
  title: 概率方法与期望线性性的随机化算法分析框架
  type: framework
  source_books: ["Mathematics for Computer Science", "Advanced Algorithms and Data Structures"]
  source_chapter: "MCS Ch.18 / AADS Ch.3–6"
  source_quote: |
    "Great Expectations... Linearity of Expectation." (MCS)
    "One of the most beautiful aspects of lottery scheduling is its use of randomness. When you have to make a decision, using such a randomized approach is often a robust and simple way of doing so." (AADS, via OSTEP reference)
  summary: |
    适用场景：随机化算法设计、概率数据结构（Bloom Filter、Treap、Skip List）、负载均衡、资源分配。
    推理步骤：
    1) 将复杂随机变量分解为指示随机变量之和；
    2) 利用期望的线性性（Linearity of Expectation），即使变量间不独立，也可分别计算期望再求和；
    3) 通过马尔可夫不等式（Markov's Theorem）或切比雪夫不等式（Chebyshev's Theorem）给出概率上界；
    4) 在算法设计中，利用随机化避免最坏情况（如快速排序的随机 pivot、Treap 的随机优先级、Lottery Scheduling 的随机票选）；
    5) 评估空间-时间-正确性的权衡：概率数据结构以可控的错误率换取显著的空间或时间节省。
    核心假设：随机源足够均匀；期望行为足以代表算法性能；小概率错误可被业务接受。
  tags: [probability-method, linearity-of-expectation, randomized-algorithms, probabilistic-data-structures]

- id: f20
  title: 概率数据结构的空间-时间-正确性权衡框架
  type: framework
  source_books: ["Advanced Algorithms and Data Structures"]
  source_chapter: "AADS Ch.4–6 — Bloom Filters, Treaps, R-trees"
  source_quote: |
    "It is important to have a good intuition about them in such a way that one can use them as building blocks to create larger projects or to solve problems." / "Probabilistic data structures offer approximate correctness and space-time tradeoffs."
  summary: |
    适用场景：海量数据去重、近似成员查询、空间索引、实时推荐系统。
    推理步骤：
    1) 明确查询类型与可接受的错误类型：假阳性（Bloom Filter）、假阴性（Count-Min Sketch）、近似最近邻（R-tree / LSH）；
    2) 根据数据规模与内存预算，选择概率数据结构而非精确结构；
    3) 用数学模型参数化结构：Bloom Filter 的位数组大小 m、哈希函数数 k 与假阳性率 p 的关系；Treap 的期望高度 O(log n)；
    4) 在系统中将概率结构作为“快速路径”：先进行低成本近似过滤，仅对候选集进行精确验证；
    5) 监控实际错误率，当分布偏离假设时动态调整参数或回退到精确结构。
    核心假设：数据分布可建模；假阳性/假阴性的代价不对称且可控；查询模式以成员查询或最近邻为主。
  tags: [probabilistic-data-structures, bloom-filter, treap, space-time-tradeoff, approximate-algorithms]

- id: f21
  title: 安全属性与活性属性的系统正确性分类框架
  type: framework
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "DDIA Chapter 8 — System Model and Reality"
  source_quote: |
    "To clarify the situation, it is worth distinguishing between two different kinds of properties: safety and liveness properties... Safety is often informally defined as nothing bad happens, and liveness as something good eventually happens."
  summary: |
    适用场景：分布式算法、并发协议、容错系统的正确性规格说明与验证。
    推理步骤：
    1) 将系统期望行为拆分为安全属性（Safety）与活性属性（Liveness）；
    2) 安全属性：定义“坏事”并证明算法在任何执行轨迹下都不会发生（如唯一性、单调序列、不可双花）；
    3) 活性属性：定义“好事”并证明在合理假设下最终会达成（如可用性、最终一致性、共识达成）；
    4) 当系统模型变化（同步/异步、崩溃恢复/拜占庭故障）时，分别检查安全与活性是否仍然保持；
    5) 安全属性通常更容易被破坏，需优先用形式化方法或模型检查验证；活性属性常依赖超时与重试机制。
    核心假设：系统模型（故障模型、时序假设）可被精确描述；安全与活性可形式化表述。
  tags: [safety, liveness, distributed-algorithms, correctness, fault-tolerance]

- id: f22
  title: 磁盘调度与电梯算法（SCAN/C-SCAN）的局部性利用框架
  type: framework
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "OSTEP Ch.37 — Disk Scheduling"
  source_quote: |
    "The algorithm, originally called SCAN, simply moves back and forth across the disk servicing requests in order across the tracks... For reasons that should now be clear, the SCAN algorithm (and its cousins) is sometimes referred to as the elevator algorithm."
  summary: |
    适用场景：机械硬盘、SSD 的 I/O 调度、日志系统、数据库存储引擎的请求重排序。
    推理步骤：
    1) 分析 I/O 请求的地址分布：若请求高度随机，调度收益有限；若存在空间局部性，调度可显著降低寻道时间；
    2) 对机械硬盘，采用 SCAN（电梯算法）或 C-SCAN 将请求按磁道排序，形成单向或双向扫描，减少磁头移动；
    3) 避免 SSTF 的饥饿问题：通过扫描方向限制，保证远端请求在下一轮扫描中被服务；
    4) 对 SSD（无机械延迟），调度重点从寻道优化转向请求合并、垃圾回收与磨损均衡；
    5) 在数据库/文件系统中，结合日志结构（LFS）或写缓冲，将随机写转化为顺序写，从根本上消除调度需求。
    核心假设：磁盘寻道时间占主导；请求到达模式可被缓冲并重排序。
  tags: [disk-scheduling, SCAN, elevator-algorithm, I/O-optimization, spatial-locality]

- id: f23
  title: 数据模型抽象与查询语言选择的决策框架
  type: framework
  source_books: ["Designing Data-Intensive Applications", "Readings in Database Systems"]
  source_chapter: "DDIA Ch.2 / RedBook Ch.1 & Ch.9"
  source_quote: |
    "Relational Versus Document Databases Today... The Object-Relational Mismatch... Graph-Like Data Models." (DDIA)
    "Nobody ever seems to learn anything from history... A decade ago, the buzz was all XML. Vendors were intent on adding XML to their relational engines." (RedBook)
  summary: |
    适用场景：数据库选型、数据平台架构、Schema 设计、多模型数据库应用。
    推理步骤：
    1) 分析数据关系模式：若实体间多为多对一/多对多，关系模型或图模型更合适；若数据自包含、层次结构明显，文档模型更自然；
    2) 评估查询模式：复杂分析查询优先选择声明式语言（SQL）与列式存储；灵活模式、快速迭代优先选择文档或键值存储；
    3) 考虑 Schema 灵活性与治理成本：强 Schema（关系型）适合数据仓库与合规场景；弱 Schema（文档/NoSQL）适合探索性数据与快速上线；
    4) 避免“重复历史”的陷阱：不为追逐技术潮流而强行将数据塞入不匹配的模型（如用 XML/JSON 存储高度关系化数据）；
    5) 在异构系统中，采用多模型共存策略：事务型数据用关系库，图遍历用图数据库，日志/缓存用键值库。
    核心假设：数据关系结构在系统生命周期内相对稳定；查询模式可被预见。
  tags: [data-modeling, query-languages, relational-vs-document, schema-design, polyglot-persistence]

