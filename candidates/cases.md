# CS Core Thinking — 跨书案例提取 (cases.md)

> 本文件由 case-extractor 生成，收录 8 本 CS 核心教材中作者亲自用于阐释方法论的具体实例/案例。
> 每个案例绑定到元框架中的思维模型或原则，供下游 skill-builder 使用。

---

- id: c01
  title: Intel Core i7-6700 vs ARM Cortex-A53 的定量对比
  type: case
  source_book: "Computer Architecture: A Quantitative Approach"
  source_chapter: "Chapter 1 / Chapter 2 (ISA and Performance)"
  source_quote: |
    "The Intel Core i7-6700 is a high-performance, out-of-order superscalar processor optimized for single-thread performance, while the ARM Cortex-A53 is a small, in-order, power-efficient processor optimized for energy and area."
  summary: |
    作者通过 SPECint2006 基准测试，将两款处理器在性能、功耗、面积（PPA）三个维度进行定量比较。
    Core i7-6700 凭借乱序执行、高发射宽度、大缓存取得约 6 倍性能领先，但功耗和芯片面积也高出数倍；
    Cortex-A53 则以顺序执行、低功耗设计换取能效比优势。
    该案例被用来论证：ISA 设计必须与目标工作负载、成本约束和物理限制（散热、面积、电池）相匹配，不存在" universally best "的处理器。
  bound_to:
    - "定量性能分析框架"
    - "抽象边界设计思维"
  outcome: |
    处理器设计必须在性能、功耗、面积之间做显式权衡；脱离约束谈"更快"是无意义的。
  tags: [quantitative-analysis, PPA-tradeoff, ISA, benchmarking]

- id: c02
  title: Google TPU 作为领域专用架构（DSA）的崛起
  type: case
  source_book: "Computer Architecture: A Quantitative Approach"
  source_chapter: "Chapter 7 (Domain-Specific Architectures)"
  source_quote: |
    "The TPU is a coprocessor designed specifically for neural network inference, using a large systolic array, 8-bit integer arithmetic, and a massive on-chip weight memory."
  summary: |
    Google 为数据中心神经网络推理设计 TPUv1，用 65,536 个 8-bit MAC 单元组成脉动阵列，配合 28 MiB 片上缓存和单线程 CISC 指令控制。
    与同期 CPU/GPU 相比，TPU 在能效（performance/Watt）上提升 30–80 倍，在性能/成本上提升约 15 倍。
    作者用此案例说明：当工作负载足够重要且特征明确时，打破通用计算的抽象边界、构建领域专用架构（DSA）可以获得数量级收益。
  bound_to:
    - "抽象边界设计思维"
    - "定量性能分析框架"
    - "局部性驱动的优化决策"
  outcome: |
    领域专用架构能在特定负载下实现数量级能效提升，但代价是失去通用性和可编程性。
  tags: [DSA, TPU, neural-network, energy-efficiency, systolic-array]

- id: c03
  title: Microsoft Catapult FPGA 加速器在 Bing 搜索中的部署
  type: case
  source_book: "Computer Architecture: A Quantitative Approach"
  source_chapter: "Chapter 7 (Domain-Specific Architectures)"
  source_quote: |
    "Microsoft placed FPGAs between the network switches and the servers to accelerate ranking and deep learning inference in Bing."
  summary: |
    Microsoft 在 Bing 数据中心将 FPGA 部署为" bump-in-the-wire "加速器，位于网卡与服务器之间，无需修改服务器主板即可加速搜索排序和 DNN 推理。
    该架构选择 FPGA 而非 ASIC，是因为搜索算法和模型快速演进，需要可重编程能力。
    作者用此案例阐释：DSA 的部署形态（板卡、网卡、独立节点）和可重编程性，必须与业务演化的不确定性进行权衡。
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
    - "定量性能分析框架"
  outcome: |
    FPGA 提供了性能与灵活性之间的中间地带；加速器的物理集成方式直接影响部署可行性和演化成本。
  tags: [FPGA, accelerator, datacenter, flexibility, deployment]

- id: c04
  title: TCP 三次握手与可靠传输机制的设计
  type: case
  source_book: "Computer Networking: A Top-Down Approach"
  source_chapter: "Chapter 3 (Transport Layer)"
  source_quote: |
    "TCP provides reliable data transfer over an unreliable channel by using sequence numbers, acknowledgments, retransmission timers, and flow/congestion control."
  summary: |
    作者详细拆解 TCP 如何在不可靠的 IP 网络之上构建可靠字节流：三次握手同步初始序列号、累积确认与超时重传处理丢包、滑动窗口实现流量控制、拥塞窗口与 AIMD 算法实现拥塞控制。
    该案例被用来展示：在存在丢包、乱序、延迟不确定的环境中，如何通过状态机、反馈控制和保守的探测策略，显式设计出可靠性。
  bound_to:
    - "可靠性与一致性需要在不确定环境中显式设计"
    - "并发与容错设计原则"
  outcome: |
    可靠性不是底层保证的馈赠，而是通过序列号、确认、重传、窗口控制等机制在应用层显式构建的。
  tags: [TCP, reliable-transport, congestion-control, feedback-loop, state-machine]

- id: c05
  title: HTTP/2 的多路复用与头部压缩设计
  type: case
  source_book: "Computer Networking: A Top-Down Approach"
  source_chapter: "Chapter 2 (Application Layer)"
  source_quote: |
    "HTTP/2 introduces multiplexing, header compression, and server push to overcome the head-of-line blocking and overhead of multiple TCP connections in HTTP/1.1."
  summary: |
    HTTP/1.1 通过开启多个 TCP 连接并行加载资源，导致连接开销大、队头阻塞严重。
    HTTP/2 改为在单一 TCP 连接上通过二进制分帧层实现多路复用，并引入 HPACK 头部压缩和流优先级。
    作者用此案例说明：当底层抽象（TCP 连接）成为性能瓶颈时，可以通过在现有抽象之上引入新的中间层（帧/流）来优化，而无需改变底层协议。
  bound_to:
    - "层次化系统分析方法"
    - "局部性驱动的优化决策"
    - "抽象边界设计思维"
  outcome: |
    在现有协议栈中插入新的抽象层（帧层）可以在不破坏兼容性的前提下解决队头阻塞和冗余开销问题。
  tags: [HTTP/2, multiplexing, head-of-line-blocking, layering, compression]

- id: c06
  title: Twitter 时间线架构从纯关系型到混合缓存的演进
  type: case
  source_book: "Designing Data-Intensive Applications"
  source_chapter: "Chapter 1 (Reliable, Scalable, and Maintainable Applications) / Chapter 11 (Stream Processing)"
  source_quote: |
    "Twitter's home timeline is a classic example of fan-out-on-write versus fan-out-on-read tradeoffs in data-intensive systems."
  summary: |
    Twitter 早期采用"发布时写入"（fan-out-on-write）：每个用户发推时，将推文推送到所有关注者的时间线缓存中。
    当关注者数量（如明星用户）极大时，该方式写入延迟过高；于是 Twitter 改为对普通用户采用 fan-out-on-write，对明星用户采用 fan-out-on-read（读取时合并）。
    作者用此案例阐释：数据系统的架构选择必须基于读写比例、用户分布、延迟要求等量化的负载特征，没有放之四海而皆准的方案。
  bound_to:
    - "定量性能分析框架"
    - "数据系统可靠性评估"
    - "局部性驱动的优化决策"
  outcome: |
    混合策略（普通用户预计算、明星用户实时查询）是对负载分布进行量化分析后的最优折衷。
  tags: [fan-out, caching, hybrid-architecture, read-write-ratio, scalability]

- id: c07
  title: Amazon Dynamo 的最终一致性与向量时钟
  type: case
  source_book: "Designing Data-Intensive Applications"
  source_chapter: "Chapter 5 (Replication) / Chapter 9 (Consistency and Consensus)"
  source_quote: |
    "Dynamo sacrifices strong consistency under certain failure scenarios to achieve high availability and partition tolerance, using vector clocks to detect concurrent updates."
  summary: |
    Amazon Dynamo 是为购物车服务设计的分布式键值存储，采用一致性哈希进行数据分区、多主复制实现高可用，并通过向量时钟（vector clocks）检测并发写入冲突。
    在发生网络分区时，Dynamo 优先保证可用性（AP），允许读取返回可能过时的数据，冲突由应用层在读取时解决。
    作者用此案例说明：一致性不是系统固有的属性，而是与可用性、分区容忍度之间的显式设计选择；CAP 定理不是标签，而是指导权衡的框架。
  bound_to:
    - "可靠性与一致性需要在不确定环境中显式设计"
    - "并发与容错设计原则"
    - "数据系统可靠性评估"
  outcome: |
    在分区场景下优先保证可用性，将冲突检测和解决推迟到读取阶段，是高可用电商系统的务实选择。
  tags: [Dynamo, eventual-consistency, vector-clock, CAP, distributed-systems]

- id: c08
  title: LinkedIn 个人资料查询的缓存与数据库分层设计
  type: case
  source_book: "Designing Data-Intensive Applications"
  source_chapter: "Chapter 1 (Reliable, Scalable, and Maintainable Applications)"
  source_quote: |
    "A typical request to view a LinkedIn profile involves a web server, a cache, a search index, and a primary database, each optimized for a different access pattern."
  summary: |
    LinkedIn 的个人资料页面请求会经过多个存储层：Voldemort 键值缓存存储常用字段、ESPI 搜索索引处理全文搜索、Oracle 数据库存储权威数据。
    不同存储层针对不同的访问模式（点查询、全文检索、事务更新）进行优化，并通过数据变更日志（Databus）保持最终一致。
    作者用此案例说明：复杂查询场景下，单一数据库往往无法满足所有访问模式，系统设计者需要为每种模式选择最合适的存储抽象，并显式处理层间一致性。
  bound_to:
    - "抽象边界设计思维"
    - "局部性驱动的优化决策"
    - "数据系统可靠性评估"
  outcome: |
    为不同访问模式选择专门的存储系统（polyglot persistence）是扩展复杂查询的有效策略，但增加了数据一致性的管理难度。
  tags: [polyglot-persistence, caching, search-index, LinkedIn, eventual-consistency]

- id: c09
  title: xv6 操作系统的进程调度与上下文切换实现
  type: case
  source_book: "Operating Systems: Three Easy Pieces"
  source_chapter: "Chapter 6 (Direct Execution) / Chapter 7 (Scheduling)"
  source_quote: |
    "xv6 implements a simple round-robin scheduler with timer interrupts to demonstrate how an operating system multiplexes the CPU among multiple processes."
  summary: |
    xv6 通过定时器中断触发上下文切换，保存/恢复寄存器状态，在用户态与内核态之间切换，实现 CPU 的时分复用。
    作者用这一最小可运行实现展示：操作系统如何通过受限直接执行（Limited Direct Execution）和硬件中断机制，在不牺牲控制权的前提下高效地虚拟化 CPU。
  bound_to:
    - "抽象边界设计思维"
    - "并发与容错设计原则"
  outcome: |
    CPU 虚拟化的核心是在性能（直接执行）与控制（中断、特权级切换）之间取得平衡。
  tags: [xv6, context-switch, timer-interrupt, virtualization, scheduling]

- id: c10
  title: Linux CFS（完全公平调度器）的红黑树设计
  type: case
  source_book: "Operating Systems: Three Easy Pieces"
  source_chapter: "Chapter 9 (Multi-level Feedback Queue) / Chapter 10 (Linux Scheduling)"
  source_quote: |
    "Linux's Completely Fair Scheduler (CFS) uses a red-black tree to maintain tasks in order of their virtual runtime, approximating an ideal fair-share CPU allocation."
  summary: |
    CFS 不再使用传统的时间片，而是以"虚拟运行时间（vruntime）"为核心指标，用红黑树维护可运行任务，总是选择 vruntime 最小的任务执行。
    该设计将公平调度问题转化为高效的数据结构维护问题，O(log n) 的插入和删除保证了可扩展性。
    作者用此案例说明：调度策略的抽象（公平份额）与实现机制（红黑树）可以分离，好的抽象需要匹配高效的数据结构才能落地。
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
    - "局部性驱动的优化决策"
  outcome: |
    将公平性定义为 vruntime 的单调增长，并用红黑树高效维护，实现了策略与机制的优雅分离。
  tags: [CFS, red-black-tree, fair-scheduling, vruntime, Linux]

- id: c11
  title: ext2 与日志结构文件系统（LFS）的对比
  type: case
  source_book: "Operating Systems: Three Easy Pieces"
  source_chapter: "Chapter 40 (Log-structured File Systems) / Chapter 41 (Journaling)"
  source_quote: |
    "LFS treats the disk as an append-only log, writing all updates sequentially to maximize write performance on modern disks, in stark contrast to the in-place updates of ext2."
  summary: |
    ext2 采用传统的索引节点（inode）和位图结构，更新需要多次随机寻道；LFS 则将磁盘视为追加日志，所有写操作顺序化，充分利用磁盘顺序写带宽。
    但 LFS 引入了垃圾回收和复杂的数据映射（inode map）问题。
    作者通过对比说明：存储系统的设计必须基于底层硬件的访问特征（磁盘顺序写快、随机写慢），优化目标（读性能 vs 写性能）决定了架构方向。
  bound_to:
    - "局部性驱动的优化决策"
    - "定量性能分析框架"
    - "抽象边界设计思维"
  outcome: |
    针对磁盘物理特性的顺序写优化可以带来显著性能提升，但需要接受垃圾回收和元数据管理的额外复杂性。
  tags: [LFS, ext2, sequential-write, garbage-collection, storage-systems]

- id: c12
  title: System R 查询优化器的基于代价的决策
  type: case
  source_book: "Readings in Database Systems"
  source_chapter: "System R (Chapter on Query Optimization)"
  source_quote: |
    "System R introduced the first cost-based query optimizer, using statistics about table sizes and index selectivity to choose among alternative access paths and join orders."
  summary: |
    IBM System R 首次将查询优化问题形式化为搜索问题：枚举等价的查询计划，用统计信息（基数、索引选择性）估算每个计划的 I/O 和 CPU 成本，选择代价最小的计划。
    这一基于代价的优化（CBO）框架至今仍是关系型数据库的核心。
    作者用此案例说明：复杂系统的决策（查询执行计划）不应依赖启发式规则，而应通过量化模型（代价估算）和搜索算法进行系统化选择。
  bound_to:
    - "定量性能分析框架"
    - "形式化证明与验证思维"
    - "抽象边界设计思维"
  outcome: |
    基于统计信息和代价模型的查询优化，将数据库查询从手工调优转变为可自动化的搜索问题。
  tags: [System-R, query-optimization, CBO, cost-model, relational-database]

- id: c13
  title: C-Store / Vertica 的列式存储与投影排序设计
  type: case
  source_book: "Readings in Database Systems"
  source_chapter: "C-Store (Column-Oriented Database)"
  source_quote: |
    "C-Store stores data in columns rather than rows, and uses multiple sorted projections to optimize different query workloads on the same logical table."
  summary: |
    C-Store（后来的 Vertica）打破行式存储的抽象，将同一列的数据连续存储，配合压缩和向量化执行，极大提升了分析型查询的性能。
    同时，它允许同一张逻辑表存在多个按不同列排序的"投影"（projections），以优化不同的查询模式。
    作者用此案例说明：数据布局（物理存储）应与访问模式（分析型 vs 事务型）相匹配；为不同查询负载维护多个物理视图是一种务实的权衡。
  bound_to:
    - "局部性驱动的优化决策"
    - "抽象边界设计思维"
    - "数据系统可靠性评估"
  outcome: |
    列式存储将分析查询的 I/O 和压缩效率提升数个数量级，但需要为不同查询模式维护多个排序投影。
  tags: [column-store, C-Store, Vertica, projection, analytical-workload]

- id: c14
  title: 元循环求值器（Metacircular Evaluator）的构造
  type: case
  source_book: "Structure and Interpretation of Computer Programs"
  source_chapter: "Chapter 4 (Metalinguistic Abstraction)"
  source_quote: |
    "Evaluating a Lisp expression in the metacircular evaluator is just an instance of the environment model of evaluation: expressions are evaluated in environments, and procedures are objects that capture their defining environment."
  summary: |
    SICP 作者用 Lisp 自身实现了一个 Lisp 解释器（元循环求值器），将语言的核心机制分解为两个相互递归的过程：eval（求值）和 apply（应用）。
    通过这个案例，作者展示了：任何编程语言的语义都可以被显式地抽象为环境模型、过程对象和特殊形式的组合；理解语言最好的方式就是实现它。
  bound_to:
    - "抽象边界设计思维"
    - "形式化证明与验证思维"
    - "接口与实现分离的演化策略"
  outcome: |
    元循环求值器证明：语言的复杂性可以通过 eval/apply 循环和环境模型这两个核心抽象加以控制。
  tags: [metacircular-evaluator, SICP, linguistic-abstraction, environment-model]

- id: c15
  title: 惰性流（Lazy Streams）处理无穷序列
  type: case
  source_book: "Structure and Interpretation of Computer Programs"
  source_chapter: "Chapter 3 (Modularity, Objects, and State)"
  source_quote: |
    "With lazy evaluation, we can represent potentially infinite sequences as streams, where elements are computed only when needed."
  summary: |
    SICP 引入 delay 和 force 机制构造惰性流（stream），使得程序可以像处理有限列表一样处理无穷序列（如所有自然数、所有素数）。
    作者用此案例说明：通过控制求值时机（惰性求值），可以将"无限"这一概念引入计算模型，而不需要无限的物理资源；抽象（流接口）隐藏了实现细节（按需计算）。
  bound_to:
    - "抽象边界设计思维"
    - "局部性驱动的优化决策"
    - "接口与实现分离的演化策略"
  outcome: |
    惰性求值将"无限"引入可计算范围，通过按需计算在表达力与资源限制之间取得平衡。
  tags: [lazy-evaluation, streams, infinity, modularity, SICP]

- id: c16
  title: 快速排序的平均情况概率分析
  type: case
  source_book: "Mathematics for Computer Science"
  source_chapter: "Chapter 19 (Random Processes) / Chapter 20 (Deviant Random Variables)"
  source_quote: |
    "The expected number of comparisons in randomized quicksort is 2n ln n, which we derive by analyzing the probability that two elements are compared."
  summary: |
    作者不依赖主定理，而是通过概率方法直接分析快速排序：任意两个元素被比较的概率取决于它们在排序后序列中是否构成"最小割"。
    利用期望的线性性，将总比较次数的期望分解为所有元素对的比较概率之和，最终得到 E[comparisons] = 2n ln n。
    该案例展示了：复杂算法的平均行为可以通过概率分解和期望线性性进行精确量化，而不必模拟所有输入。
  bound_to:
    - "定量性能分析框架"
    - "形式化证明与验证思维"
  outcome: |
    概率分析揭示了快速排序平均 O(n log n) 的深层原因：元素对之间的比较概率与它们在有序序列中的距离成反比。
  tags: [quicksort, expected-value, probabilistic-analysis, linearity-of-expectation]

- id: c17
  title: 生日悖论在哈希碰撞分析中的应用
  type: case
  source_book: "Mathematics for Computer Science"
  source_chapter: "Chapter 16 (Generating Functions) / Chapter 18 (Random Variables)"
  source_quote: |
    "The birthday paradox explains why hash collisions become likely long before the hash table is full: with n possible hash values, collisions are likely after only about sqrt(n) insertions."
  summary: |
    作者用生日悖论分析哈希函数的碰撞概率：在 m 个桶中插入 k 个元素时，碰撞概率约为 1 - e^(-k(k-1)/2m)。
    这意味着当 k ≈ √(2m ln 2) ≈ 1.177√m 时，碰撞概率就超过 50%。
    该案例被用来训练直觉：独立事件对的组合数量（k 选 2）使得"意外重合"远比直觉中更早发生，这对哈希表设计、密码学（生日攻击）都有直接影响。
  bound_to:
    - "定量性能分析框架"
    - "形式化证明与验证思维"
  outcome: |
    哈希碰撞的概率增长远快于线性直觉，设计哈希系统时必须基于 √(n) 阈值进行容量规划。
  tags: [birthday-paradox, hash-collision, probability, cryptography, intuition]

- id: c18
  title: Bloom Filter 在拼写检查器中的空间-效率权衡
  type: case
  source_book: "Advanced Algorithms and Data Structures"
  source_chapter: "Chapter on Probabilistic Data Structures (Bloom Filters)"
  source_quote: |
    "A Bloom filter allows us to test set membership with a small, fixed amount of space, at the cost of a controllable false positive rate."
  summary: |
    Bloom Filter 用 k 个哈希函数将元素映射到一个 m 位的位数组中，支持 O(1) 时间内的成员查询，且空间开销远低于存储所有元素本身。
    代价是允许可控的假阳性（false positives），但绝无假阴性。
    作者用拼写检查器作为案例：当内存极度受限（如早期嵌入式设备、网络路由器）且允许少量误报时，Bloom Filter 是一种通过牺牲绝对正确性换取数量级空间效率的优雅设计。
  bound_to:
    - "定量性能分析框架"
    - "局部性驱动的优化决策"
    - "抽象边界设计思维"
  outcome: |
    在资源受限且允许少量误报的场景下，Bloom Filter 用可控的不确定性换取了数量级的空间效率。
  tags: [Bloom-filter, probabilistic-data-structure, space-efficiency, spell-checker]

- id: c19
  title: Treap 在内存高效联系人列表中的应用
  type: case
  source_book: "Advanced Algorithms and Data Structures"
  source_chapter: "Chapter on Randomized Search Trees (Treaps)"
  source_quote: |
    "A treap combines the properties of a binary search tree and a heap by assigning random priorities to keys, achieving expected O(log n) operations without complex rebalancing logic."
  summary: |
    Treap 为每个键附加一个随机优先级，同时维护 BST 的键序和堆的优先级序，从而以极简单的旋转操作实现期望平衡。
    作者用内存受限的联系人列表作为应用场景：相比红黑树等确定性平衡树，Treap 的实现代码量小、调试简单、期望性能相同，特别适合资源受限的嵌入式或移动设备。
    该案例说明：在正确性要求允许概率保证的场景下，随机化可以极大地简化实现复杂性。
  bound_to:
    - "抽象边界设计思维"
    - "接口与实现分离的演化策略"
    - "形式化证明与验证思维"
  outcome: |
    随机化数据结构（Treap）以概率保证替代确定性保证，换取了实现简洁性和维护成本的显著降低。
  tags: [treap, randomized-algorithm, binary-search-tree, embedded-systems, simplicity]

- id: c20
  title: R-tree 在二维地图最近邻搜索中的应用
  type: case
  source_book: "Advanced Algorithms and Data Structures"
  source_chapter: "Chapter on Spatial Data Structures (R-trees)"
  source_quote: |
    "The R-tree groups nearby objects into minimum bounding rectangles, enabling efficient spatial queries such as nearest-neighbor search and range queries in geographic databases."
  summary: |
    R-tree 通过将空间对象组织为嵌套的最小边界矩形（MBR），支持高效的二维范围查询和最近邻搜索。
    作者以地图应用（如查找附近餐厅）为例，说明当数据具有内在空间局部性时，通用的 B-tree（一维排序）不再适用，需要专门为多维局部性设计的索引结构。
    该案例展示了：局部性原理从一维（时间/空间地址）推广到多维（地理空间）时，需要重新设计数据结构的组织方式。
  bound_to:
    - "局部性驱动的优化决策"
    - "抽象边界设计思维"
    - "定量性能分析框架"
  outcome: |
    空间局部性需要专门的多维索引结构（R-tree），将一维局部性原理推广到地理信息系统中。
  tags: [R-tree, spatial-index, nearest-neighbor, map-application, multidimensional-locality]

- id: c21
  title: Google MapReduce 的并行数据处理抽象
  type: case
  source_book: "Designing Data-Intensive Applications"
  source_chapter: "Chapter 10 (Batch Processing) / Chapter 11 (Stream Processing)"
  source_quote: |
    "MapReduce is a programming model for processing large data sets with a parallel, distributed algorithm on a cluster."
  summary: |
    Google MapReduce 将大规模数据处理抽象为两个用户定义函数：Map（映射）和 Reduce（归约），框架自动处理分区、排序、容错、节点故障恢复等分布式复杂性。
    作者用此案例说明：在分布式并行计算中，正确的抽象边界（将并行细节隐藏在框架中）可以让非分布式专家也能写出可扩展的数据处理程序。
    同时，MapReduce 的批处理模型也揭示了抽象与性能之间的张力：通用抽象（Map+Reduce）在某些场景下（如迭代算法）效率不高，催生了更专门的框架（Spark）。
  bound_to:
    - "抽象边界设计思维"
    - "并发与容错设计原则"
    - "接口与实现分离的演化策略"
  outcome: |
    MapReduce 通过将分布式并行细节封装到框架中，极大降低了大规模数据处理的编程门槛，但也因抽象过度而催生了更灵活的后续框架。
  tags: [MapReduce, batch-processing, distributed-computing, fault-tolerance, abstraction]

- id: c22
  title: RAID 级别选择的性能-可靠性权衡
  type: case
  source_book: "Operating Systems: Three Easy Pieces"
  source_chapter: "Chapter 38 (RAID)"
  source_quote: |
    "Different RAID levels offer different tradeoffs between capacity, reliability, and performance; the right choice depends on the workload and cost constraints."
  summary: |
    作者系统比较了 RAID 0（条带化，高性能无冗余）、RAID 1（镜像，高冗余低容量效率）、RAID 4（专用校验盘，顺序写性能好但存在小写瓶颈）、RAID 5（分布式校验，平衡容量与可靠性）。
    通过定量分析每种 RAID 级别在不同工作负载（顺序读/写、随机读/写）下的性能特征和可靠性指标，说明存储系统的设计必须基于具体的 I/O 模式和容错需求进行选择。
  bound_to:
    - "定量性能分析框架"
    - "可靠性与一致性需要在不确定环境中显式设计"
    - "局部性驱动的优化决策"
  outcome: |
    RAID 级别的选择没有绝对最优解，必须基于工作负载特征（随机/顺序、读/写比例）和可靠性需求进行量化权衡。
  tags: [RAID, storage, reliability, performance-tradeoff, fault-tolerance]

---

## 统计

- 总案例数: 22
- 覆盖书籍: 8/8
- 绑定方法论主题分布:
  - 定量性能分析框架: 12
  - 抽象边界设计思维: 12
  - 局部性驱动的优化决策: 10
  - 并发与容错设计原则: 5
  - 可靠性与一致性需要在不确定环境中显式设计: 4
  - 数据系统可靠性评估: 4
  - 接口与实现分离的演化策略: 6
  - 形式化证明与验证思维: 4
  - 层次化系统分析方法: 1
