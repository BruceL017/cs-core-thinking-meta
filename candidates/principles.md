- id: p01
  title: 优先优化常见情况
  type: principle
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "1.9 Quantitative Principles of Computer Design"
  source_quote: |
    "Perhaps the most important and pervasive principle of computer design is to focus on the common case: in making a design trade-off, favor the frequent case over the infrequent case. This principle applies when determining how to spend resources, because the impact of the improvement is higher if the occurrence is commonplace."
  summary: |
    在做设计权衡时，永远优先投资常见情况（frequent case），而不是罕见情况。优化前必须先量化测量目标特征在实际执行中所占的时间比例；如果不对使用频率进行测量就投入大量优化精力，往往会落入 Amdahl 定律的陷阱，导致整体加速比令人失望。
  tags: [performance, optimization, measurement, amdalh]

- id: p02
  title: 永远不要忽略不可加速部分的比例
  type: principle
  source_books: ["Computer Architecture: A Quantitative Approach"]
  source_chapter: "1.9 Quantitative Principles of Computer Design"
  source_quote: |
    "Virtually every practicing computer architect knows Amdahl’s Law. Despite this, we almost all occasionally expend tremendous effort optimizing some feature before we measure its usage. Only when the overall speedup is disappointing do we recall that we should have measured first before we spent so much effort enhancing it!"
  summary: |
    在评估任何加速方案时，必须先测量被优化部分在整体执行时间中的真实占比。Amdahl 定律指出：整体加速比受限于不可加速部分的比重。忽略这一点会导致资源浪费和失望的回报。
  tags: [performance, amdalh, quantification]

- id: p03
  title: 当数据访问呈现强局部性时，优先投资缓存与层次化存储
  type: principle
  source_books: ["Computer Architecture: A Quantitative Approach", "Advanced Algorithms and Data Structures"]
  source_chapter: "1.9 Principle of Locality / Chapter 2 d-ary heaps"
  source_quote: |
    "The principle of locality, presented in the first chapter, says that most programs do not access all code or data uniformly. Locality occurs in time (temporal locality) and in space (spatial locality)." / "When the size of the heap is larger than available cache or than the available memory, or in any case where caching and multiple levels of storage are involved, then on average a binary heap requires more cache misses or page faults than a d-ary heap."
  summary: |
    90/10 局部性规则指出程序约 90% 的执行发生在 10% 的代码上，数据访问也具有时间/空间局部性。此时应优先通过缓存、预取、调整分支因子或层次化存储来降低平均内存访问时间，而不是单纯增加原始带宽。
  tags: [locality, cache, memory-hierarchy, optimization]

- id: p04
  title: 端到端功能应在最高层实现
  type: principle
  source_books: ["Computer Networking: A Top-Down Approach", "Designing Data-Intensive Applications"]
  source_chapter: "3.3 UDP / 8.1 Partial Failures"
  source_quote: |
    "This is an example of the celebrated end-end principle in system design [Saltzer 1984], which states that since certain functionality (error detection, in this case) must be implemented on an end-end basis: 'functions placed at the lower levels may be redundant or of little value when compared to the cost of providing them at the higher level.'"
  summary: |
    如果某项功能（如错误检测、可靠性保证、一致性校验）必须在端到端之间正确完成，那么它应该在通信的端系统（最高层）实现，而不是在底层网络中重复或依赖底层保证。底层优化不能替代端到端的显式处理。
  tags: [end-to-end, reliability, distributed-systems, layering]

- id: p05
  title: 在不可靠信道上实现可靠传输的黄金法则
  type: principle
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "3.4 Principles of Reliable Data Transfer"
  source_quote: |
    "Fundamentally, three additional protocol capabilities are required in ARQ protocols to handle the presence of bit errors: Error detection... Receiver feedback... Retransmission." / "A complete and reliable data transfer protocol requires the addition of sequence numbers and timers."
  summary: |
    当底层信道可能丢包、乱序或损坏比特时，可靠传输协议必须显式组合五种机制：差错检测（checksum）、接收方反馈（ACK/NAK）、序列号（sequence numbers）、定时器（timers）和重传（retransmission）。缺一不可。
  tags: [reliable-transfer, networking, fault-tolerance, protocols]

- id: p06
  title: 拥塞控制应收敛到公平带宽共享
  type: principle
  source_books: ["Computer Networking: A Top-Down Approach"]
  source_chapter: "3.7 TCP Congestion Control"
  source_quote: |
    "A congestion-control mechanism is said to be fair if the average transmission rate of each connection is approximately R/K; that is, each connection gets an equal share of the link bandwidth." / "[Chiu 1989] provides an elegant and intuitive explanation of why TCP congestion control converges to provide an equal share of a bottleneck link’s bandwidth among competing TCP connections."
  summary: |
    当多个连接共享同一瓶颈链路时，拥塞控制算法应使各连接的平均传输速率收敛到链路带宽的近似均等份额（R/K）。AIMD（加性增、乘性减）机制在理想条件下具有这种收敛性；设计新的传输协议时必须显式考虑公平性，防止 UDP 或多并行连接抢占带宽。
  tags: [congestion-control, fairness, networking, tcp]

- id: p07
  title: 用容错机制将不可靠组件构建成可靠系统
  type: principle
  source_books: ["Designing Data-Intensive Applications", "Operating Systems: Three Easy Pieces", "Readings in Database Systems"]
  source_chapter: "1.1 Reliability / 42 Crash Consistency / Chapter 5 Transactions"
  source_quote: |
    "It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. In this book we cover several techniques for building reliable systems from unreliable parts."
  summary: |
    故障无法被完全消除，因此不要试图通过完美预防来实现可靠性。相反，应设计显式的容错（fault-tolerance）机制，使得单个组件的故障不会升级为系统级失效。这一原则在数据库（WAL/ARIES）、操作系统（日志/FSCK）和分布式系统（复制/共识）中通用。
  tags: [reliability, fault-tolerance, distributed-systems, databases]

- id: p08
  title: 主动注入故障以验证容错机制
  type: principle
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "1.1 Reliability"
  source_quote: |
    "Counterintuitively, in such fault-tolerant systems, it can make sense to increase the rate of faults by triggering them deliberately—for example, by randomly killing individual processes without warning... by deliberately inducing faults, you ensure that the fault-tolerance machinery is continually exercised and tested, which can increase your confidence that faults will be handled correctly when they occur naturally."
  summary: |
    在已经具备容错能力的系统中，应有意识地、随机地触发故障（如 Chaos Monkey），以持续检验容错路径是否真正可用。被动等待真实故障会掩盖错误处理代码中的缺陷；主动故障注入是验证分布式系统韧性的必要实践。
  tags: [chaos-engineering, fault-tolerance, testing, reliability]

- id: p09
  title: 不信任绝对保证，持续审计验证
  type: principle
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "12.3 Trust, but Verify"
  source_quote: |
    "However, not many systems currently have this kind of 'trust, but verify' approach of continually auditing themselves. Many assume that correctness guarantees are absolute and make no provision for the possibility of rare data corruption. I hope that in the future we will see more self-validating or self-auditing systems that continually check their own integrity, rather than relying on blind trust."
  summary: |
    不要对硬件、网络校验和或数据库事务的绝对正确性抱有盲目信任。应建立持续审计（auditing）机制：定期读取并校验数据、从备份恢复测试、检测静默损坏。"信任但验证"是长期运行数据密集型系统的安全底线。
  tags: [auditing, data-integrity, verification, trust-but-verify]

- id: p10
  title: 可扩展性讨论必须以量化负载描述为前提
  type: principle
  source_books: ["Designing Data-Intensive Applications"]
  source_chapter: "1.2 Scalability"
  source_quote: |
    "There is unfortunately no easy fix for making applications reliable, scalable, or maintainable. Rather, we will try to think about systems with operability, simplicity, and evolvability in mind." / "It is meaningless to say 'X is scalable' or 'Y doesn’t scale.' Rather, discussing scalability means considering questions such as 'If the system grows in a particular way, what are our options for coping with the growth?'"
  summary: |
    空谈"某系统可扩展"没有意义。在评估扩展性之前，必须先用具体指标（如请求速率、数据量、读写比例、延迟分布）对负载进行量化描述，并定义当负载翻倍时系统的应对策略（垂直扩展、水平扩展、分片、缓存等）。
  tags: [scalability, load-modeling, quantification, performance]

- id: p11
  title: 可维护性 = 可操作性 + 简单性 + 可演化性
  type: principle
  source_books: ["Designing Data-Intensive Applications", "Structure and Interpretation of Computer Programs"]
  source_chapter: "1.3 Maintainability / Preface"
  source_quote: |
    "Good operability means making routine tasks easy, allowing the operations team to focus their efforts on high-value activities." / "Reducing complexity greatly improves the maintainability of software, and thus simplicity should be a key goal for the systems we build." / "The ease with which you can modify a data system... is closely linked to its simplicity and its abstractions."
  summary: |
    长期运行的系统必须把可维护性分解为三个可操作维度：可操作性（让运维团队轻松完成日常任务）、简单性（通过抽象消除偶然复杂度）、可演化性（接口与实现分离，使未来变更容易）。好的抽象是同时提升这三者的核心杠杆。
  tags: [maintainability, operability, simplicity, evolvability, abstraction]

- id: p12
  title: 通过虚拟化抽象实现资源共享
  type: principle
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "2.1 Virtualization / 3 A Dialogue on Virtualization"
  source_quote: |
    "How does the OS do so efficiently? What hardware support is required? The answer is virtualization. That is, the OS takes a physical resource (such as the processor, or memory, or a disk) and transforms it into a more general, powerful, and easy-to-use virtual form of itself."
  summary: |
    当多个程序需要共享同一物理资源（CPU、内存、磁盘）时，应引入虚拟化抽象，使每个程序都看到一份独立、通用、易于使用的虚拟资源。操作系统通过 CPU 虚拟化（时间共享）和内存虚拟化（地址空间隔离）实现安全高效的资源共享。
  tags: [virtualization, resource-sharing, os, abstraction]

- id: p13
  title: 机制与策略分离
  type: principle
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "5.4 Why? Motivating The API"
  source_quote: |
    "Well, as it turns out, the separation of fork() and exec() is essential in building a UNIX shell, because it lets the shell run code after the call to fork() but before the call to exec(); this code can alter the environment of the about-to-be-run program, and thus enables a variety of interesting features to be readily built."
  summary: |
    设计操作系统接口或资源管理器时，应将"机制"（how，如进程创建、内存分配、I/O 传输）与"策略"（what/which，如调度算法、页面置换策略、权限检查）分离。这种分离使同一机制可以支撑多种策略，增强系统的灵活性和可扩展性。
  tags: [mechanism-policy-separation, os, design, flexibility]

- id: p14
  title: 并发访问共享资源时必须显式同步并避免循环等待
  type: principle
  source_books: ["Operating Systems: Three Easy Pieces"]
  source_chapter: "26 Concurrency: An Introduction / 32 Common Concurrency Problems"
  source_quote: |
    "A critical section is a piece of code that accesses a shared variable (or more generally, a shared resource) and must not be concurrently executed by more than one thread." / "Probably the most practical prevention technique... is to write your locking code such that you never induce a circular wait. The most straightforward way to do that is to provide a total ordering on lock acquisition."
  summary: |
    当多个执行流访问共享资源时，必须使用锁或其他同步原语建立互斥（mutual exclusion），防止竞态条件（race condition）。如果系统需要同时持有多个锁，必须定义全局或局部的锁获取顺序（total/partial ordering），以消除循环等待、避免死锁。
  tags: [concurrency, locks, deadlock-prevention, synchronization]

- id: p15
  title: 持久化数据结构更新必须通过预写日志保证崩溃一致性
  type: principle
  source_books: ["Operating Systems: Three Easy Pieces", "Designing Data-Intensive Applications", "Readings in Database Systems"]
  source_chapter: "42 Crash Consistency: FSCK and Journaling / 3.4 B-tree optimizations / Chapter 5 Transactions"
  source_quote: |
    "Probably the most popular solution to the consistent update problem is to steal an idea from the world of database management systems. That idea, known as write-ahead logging... The basic idea is as follows. When updating the disk, before overwriting the structures in place, first write down a little note... describing what you are about to do." / "The canonical algorithm for implementing a 'No Force, Steal' WAL-based recovery manager is IBM’s ARIES algorithm."
  summary: |
    在更新磁盘上的持久化数据结构时，崩溃可能发生在任意两次写操作之间。必须通过写前日志（Write-Ahead Logging / Journaling）先记录意图再执行原地更新，以便崩溃后通过重放日志快速恢复一致性。为平衡性能与安全，通常优先对元数据（metadata）做日志，而非全部用户数据。
  tags: [crash-consistency, wal, journaling, durability, databases]

- id: p16
  title: 数据库系统必须保持逻辑独立与物理独立
  type: principle
  source_books: ["Readings in Database Systems", "Designing Data-Intensive Applications"]
  source_chapter: "Chapter 3: Query Optimization / Chapter 2: Data Models and Query Languages"
  source_quote: |
    "Query optimization is important in relational database architecture because it is core to enabling data-independent query processing. Selinger et al.’s foundational paper on System R enables practical query optimization by decomposing the problem into three distinct subproblems: cost estimation, relational equivalences that define a search space, and cost-based search."
  summary: |
    数据库的核心价值之一是将数据的逻辑组织（模式、查询语言）与物理存储（索引、布局、压缩）解耦。查询优化器应基于成本模型（cost estimation）而非纯启发式规则选择执行计划，使得物理存储的变更不会影响应用程序的正确性。
  tags: [data-independence, query-optimization, cost-based, databases]

- id: p17
  title: 并发事务应优先使用可串行化隔离
  type: principle
  source_books: ["Readings in Database Systems", "Designing Data-Intensive Applications"]
  source_chapter: "Chapter 5: Concurrency Control / Chapter 7: Transactions"
  source_quote: |
    "Classically, database systems used serializable transactions as a means of enforcing consistency: if individual transactions each leave the database in a 'consistent' state, then a serializable execution... will guarantee that all transactions observe a 'consistent' state of the database." / "However, serializability is often considered too expensive to enforce."
  summary: |
    当多个事务并发访问共享数据时，可串行化（serializable）隔离是保证数据库一致性的黄金标准。如果出于性能考虑使用更弱的隔离级别（如 Read Committed、Snapshot Isolation），开发者必须显式地理解并维护应用级不变量，否则会出现丢失更新、写偏斜等异常。
  tags: [transactions, acid, serializability, isolation, databases]

- id: p18
  title: 程序必须首先为人编写，其次为机器执行
  type: principle
  source_books: ["Structure and Interpretation of Computer Programs", "Designing Data-Intensive Applications"]
  source_chapter: "Preface / 1.3 Simplicity"
  source_quote: |
    "Thus, programs must be written for people to read, and only incidentally for machines to execute." / "One of the best tools we have for removing accidental complexity is abstraction. A good abstraction can hide a great deal of implementation detail behind a clean, simple-to-understand façade."
  summary: |
    代码的主要读者是人类而非编译器。应通过清晰的抽象、一致的命名、分层设计和数据抽象屏障来控制智力复杂度。简单性不是减少功能，而是消除偶然复杂度（accidental complexity），使系统更易于理解、修改和演化。
  tags: [readability, abstraction, simplicity, software-design]

- id: p19
  title: 组合手段必须满足闭包性质
  type: principle
  source_books: ["Structure and Interpretation of Computer Programs"]
  source_chapter: "2.2 Hierarchical Data and the Closure Property"
  source_quote: |
    "In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation. Closure is the key to power in any means of combination because it permits us to create hierarchical structures—structures made up of parts, which themselves are made up of parts, and so on."
  summary: |
    设计数据或过程的组合手段时，应确保其满足闭包性质（closure property）：组合的结果可以继续被同一手段再次组合。这是构建分层、递归、可扩展复杂系统的关键——从 Lisp 的 cons 到现代函数式组件组合均遵循此原则。
  tags: [closure-property, composition, abstraction, hierarchical-design]

- id: p20
  title: 在实现前先通过证明或不变量建立正确性
  type: principle
  source_books: ["Mathematics for Computer Science", "Structure and Interpretation of Computer Programs"]
  source_chapter: "1 What is a Proof? / 5.4 State Machines"
  source_quote: |
    "Proofs play a central role in this work because the authors share a belief with most mathematicians that proofs are essential for genuine understanding. Proofs also play a growing role in computer science; they are used to certify that software and hardware will always behave correctly, something that no amount of testing can do."
  summary: |
    对于关键算法或系统属性，应在编码或部署前构造形式化证明，而不是仅依赖测试。证明能够 certify 软硬件在所有情况下行为正确，而测试只能覆盖有限样本。将数学推理前置是避免灾难性缺陷的根本方法。
  tags: [proof, correctness, formal-methods, verification]

- id: p21
  title: 寻找并维护状态机的不变量
  type: principle
  source_books: ["Mathematics for Computer Science", "Operating Systems: Three Easy Pieces"]
  source_chapter: "5.4.3 The Invariant Principle"
  source_quote: |
    "The Invariant Principle: If a preserved invariant of a state machine is true for the start state, then it is true for all reachable states. The Invariant Principle is nothing more than the Induction Principle reformulated in a convenient form for state machines."
  summary: |
    分析有状态系统或算法时，应寻找一个 preserved invariant（在状态转移下保持为真的谓词）。如果它在初始状态为真，则对所有可达状态都为真。这是验证并发协议、分布式共识和循环算法部分正确性的核心工具。
  tags: [invariant, state-machine, induction, correctness, concurrency]

- id: p22
  title: 用概率方法处理不确定性并证明存在性
  type: principle
  source_books: ["Mathematics for Computer Science", "Advanced Algorithms and Data Structures"]
  source_chapter: "18 Random Variables / Chapter 4 Bloom Filters"
  source_quote: |
    "This problem is an instance of the probabilistic method. It uses probability to prove the existence of an object without constructing it." / "Expected values obey a simple, very helpful rule called Linearity of Expectation. Its simplest form says that the expected value of a sum of random variables is the sum of the expected values of the variables."
  summary: |
    面对不确定性或组合爆炸时，应使用概率论工具（期望线性性、条件概率、指示随机变量、概率方法）进行定量分析。概率方法可以在不构造具体对象的情况下证明其存在性；期望计算可以帮助我们在海量数据场景下做出可证明的近似决策。
  tags: [probability, randomized-algorithms, expectation, uncertainty]

- id: p23
  title: 在允许少量误差时，用概率数据结构换取巨大空间节省
  type: principle
  source_books: ["Advanced Algorithms and Data Structures", "Computer Architecture: A Quantitative Approach"]
  source_chapter: "Chapter 4 Bloom Filters / Appendix B Rules of Thumb"
  source_quote: |
    "Bloom filters require less memory in comparison to hash tables; this is the main reason for their use. While a negative answer is 100% accurate, there might be false positives." / "There is a tradeoff between the accuracy of a Bloom filter and the memory it uses. The less memory, the more false positives it returns."
  summary: |
    当应用场景能够容忍有界假阳性（false positives）且不需要删除操作时，应使用 Bloom filter 等概率数据结构。它用固定数量的比特（k 位/元素）和摊销常数时间实现成员查询，相比精确结构可节省数量级的内存，但必须在设计时显式选择可接受的误差率与空间预算。
  tags: [probabilistic-data-structures, bloom-filter, space-efficiency, approximation]

- id: p24
  title: 选择数据结构时必须同时考虑渐近复杂度和缓存局部性
  type: principle
  source_books: ["Advanced Algorithms and Data Structures", "Computer Architecture: A Quantitative Approach"]
  source_chapter: "2.9 Branching Factor vs Memory / 2.10 Performance Analysis"
  source_quote: |
    "Changing the branching factor of a heap won’t affect asymptotic running time for its methods, but will still provide a constant factor improvement that matters when we move from pure theory to applications with a massive amount of data to manage." / "The larger the branching factor becomes, the more the heap becomes short and wide, and the more the principle of locality applies."
  summary: |
    在大规模数据场景下，不能仅依据大 O 渐近复杂度选择数据结构。必须同时评估内存层次结构（cache、页表、TLB）中的实际行为：分支因子、数据布局的连续性、缓存未命中率和页错误数量。有时一个渐近等价但局部性更好的结构会在实践中显著更快。
  tags: [asymptotic-analysis, cache-locality, memory-hierarchy, data-structures]
