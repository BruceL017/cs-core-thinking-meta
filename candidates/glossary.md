# CS Core Thinking Meta-Framework — 跨书共享术语词典

> 本词典由 glossary-extractor 从 8 本 CS 核心教材中提炼，重点记录作者在书中的特定用法、与常识的差异，以及下游 skill 必须掌握该澄清的原因。

---

## 体系结构与性能

- id: g01
  term: Virtualization (虚拟化)
  type: term
  source_books: ["Operating Systems: Three Easy Pieces", "Computer Architecture"]
  author_definition: |
    OSTEP 中，虚拟化是将物理资源（CPU、内存、磁盘）转化为更通用、更易于多道程序共享的抽象形式的核心机制。CPU 虚拟化通过时分复用和上下文切换实现，内存虚拟化通过地址翻译和页表实现。Hennessy & Patterson 则在体系结构层面讨论虚拟化对性能和安全的影响。
  key_distinction: |
    常识中"虚拟化"几乎等同于 VMware/KVM 等虚拟机技术；但在 OSTEP 中，虚拟化是操作系统三大主题之首，涵盖 CPU、内存、存储的广义抽象，是操作系统作为"资源管理器"的本质手段。
  why_it_matters: |
    下游 skill 若将虚拟化窄化为"跑虚拟机"，会遗漏操作系统设计的核心思维：如何通过抽象和复用，在共享与隔离之间取得平衡。
  tags: [abstraction, os, resource-management, performance]

- id: g02
  term: Quantitative Approach (定量方法)
  type: term
  source_books: ["Computer Architecture"]
  author_definition: |
    Hennessy & Patterson 强调用具体公式、基准测试、成本模型和能耗分析来评估设计决策。核心指标包括 CPI、IPC、加速比（Amdahl's Law）和功耗墙（Power Wall）。
  key_distinction: |
    常识中性能讨论常停留在"更快"或"更慢"的感性判断；而定量方法要求将性能转化为可测量、可比较、可权衡的数学表达。
  why_it_matters: |
    下游 skill 必须教会学习者用数据说话，避免"感觉优化"。没有量化就没有设计决策的理性基础。
  tags: [performance, measurement, design-tradeoff]

- id: g03
  term: Amdahl's Law (阿姆达尔定律)
  type: term
  source_books: ["Computer Architecture", "Advanced Algorithms and Data Structures"]
  author_definition: |
    系统整体加速比受限于可加速部分所占的比例。公式表达为：Speedup = 1 / [(1 - f) + f/s]。它揭示了并行化和优化的天花板。
  key_distinction: |
    常识中容易误以为"加更多核就能无限加速"；Amdahl's Law 明确指出，若串行部分占比高，硬件投入回报将急剧递减。
  why_it_matters: |
    下游 skill 在讨论性能优化和并行化时必须引入此定律，否则学习者会陷入"盲目堆硬件"的思维陷阱。
  tags: [performance, parallelism, scalability, law]

- id: g04
  term: Memory Hierarchy (存储层次)
  type: term
  source_books: ["Computer Architecture", "Operating Systems: Three Easy Pieces"]
  author_definition: |
    由寄存器、缓存、主存、磁盘等组成的金字塔结构，每一层在速度、容量、成本之间权衡。局部性原理是其存在的理论基础。
  key_distinction: |
    常识中可能将存储层次视为硬件细节；但在这些书中，它是理解系统性能、优化算法和数据结构设计的核心框架。
  why_it_matters: |
    下游 skill 若不理解存储层次，就无法解释为什么算法复杂度相同但常数差异巨大，也无法做出合理的缓存友好设计。
  tags: [performance, locality, hardware, optimization]

- id: g05
  term: Principle of Locality (局部性原理)
  type: term
  source_books: ["Computer Architecture", "Operating Systems: Three Easy Pieces", "Advanced Algorithms and Data Structures"]
  author_definition: |
    程序和数据访问在时间和空间上倾向于聚集。时间局部性指近期访问的数据很可能再次被访问；空间局部性指相邻数据很可能被连续访问。
  key_distinction: |
    常识中"局部性"可能被理解为地理或概念上的邻近；在 CS 中，它是预测访问模式、设计缓存和预取策略的形式化基础。
  why_it_matters: |
    局部性是存储层次、网络缓存、分页、算法分治等几乎所有性能优化的底层假设。下游 skill 必须将其作为第一性原理传授。
  tags: [performance, locality, caching, first-principles]

- id: g06
  term: ILP / TLP / DLP
  type: term
  source_books: ["Computer Architecture"]
  author_definition: |
    ILP（指令级并行）通过流水线、乱序执行在单核内挖掘并行；TLP（线程级并行）通过多核/多线程实现；DLP（数据级并行）通过 SIMD/GPU 对大量数据执行相同操作。
  key_distinction: |
    常识中"并行"往往笼统地等同于"多线程"或"多核"；这三者区分了不同层次的并行机制，对应完全不同的硬件支持和编程模型。
  why_it_matters: |
    下游 skill 在讨论并行优化时必须区分并行层次，否则学习者会错误地将 SIMD 问题用多线程解决，或将 ILP 潜力浪费在粗粒度并行上。
  tags: [parallelism, architecture, performance, hardware]

- id: g07
  term: Domain-Specific Architecture (DSA)
  type: term
  source_books: ["Computer Architecture"]
  author_definition: |
    为特定应用领域（如深度学习、图计算、视频编解码）定制的硬件架构。通过牺牲通用性换取能效比和性能的数量级提升。GPU 和 TPU 是典型的 DSA。
  key_distinction: |
    常识中硬件通常被分为"通用 CPU"和"专用 ASIC"两类；DSA 位于两者之间，是软件-硬件协同设计的新范式。
  why_it_matters: |
    下游 skill 需要让学习者理解：在摩尔定律放缓的背景下，DSA 是性能扩展的主要方向，系统设计者必须具备软硬件协同思维。
  tags: [architecture, hardware, design-tradeoff, specialization]

---

## 网络与通信

- id: g08
  term: Protocol Stack (协议栈)
  type: term
  source_books: ["Computer Networking", "Operating Systems: Three Easy Pieces"]
  author_definition: |
    计算机网络通过分层抽象（应用层、传输层、网络层、链路层、物理层）将复杂通信问题分解为可管理的模块。每一层通过定义良好的接口为上层提供服务，同时隐藏实现细节。
  key_distinction: |
    常识中"协议"常被理解为具体规则或报文格式；协议栈强调的是层次化抽象和关注点分离的系统性思维。
  why_it_matters: |
    下游 skill 必须传递的不仅是各层协议的功能，更是"如何通过分层抽象控制复杂性"的设计方法论。
  tags: [abstraction, networking, modularity, design-pattern]

- id: g09
  term: End-to-End Principle (端到端原则)
  type: term
  source_books: ["Computer Networking"]
  author_definition: |
    由 Saltzer、Reed、Clark 提出：通信系统的某些功能（如可靠性、完整性检查）只有在通信端点才能真正正确实现，网络核心应保持简单。
  key_distinction: |
    常识中可能认为"网络应该提供一切保障"；端到端原则指出，将功能下推到网络中间节点不仅低效，而且可能错误。
  why_it_matters: |
    这是互联网设计的核心哲学，也是 TCP 存在的基础。下游 skill 若不理解此原则，就无法解释为什么互联网采用"尽力交付"模型。
  tags: [networking, design-principle, reliability, internet]

- id: g10
  term: TCP Congestion Control (TCP 拥塞控制)
  type: term
  source_books: ["Computer Networking"]
  author_definition: |
    TCP 通过慢启动、拥塞避免、快速重传/恢复等算法，动态探测网络可用带宽并在拥塞时退让。其核心是"将网络视为共享资源，主动避免崩溃"。
  key_distinction: |
    常识中容易将 TCP 的可靠性仅理解为"丢包重传"；拥塞控制实际上是分布式资源协调算法，是互联网规模扩展的关键。
  why_it_matters: |
    下游 skill 需要澄清：TCP 不仅是传输协议，更是一个分布式反馈控制系统。理解拥塞控制对设计高吞吐量系统至关重要。
  tags: [networking, distributed-systems, control-theory, performance]

- id: g11
  term: RTT / Throughput / Packet Loss
  type: term
  source_books: ["Computer Networking", "Designing Data-Intensive Applications"]
  author_definition: |
    RTT（往返时间）是网络延迟的基本度量；Throughput（吞吐量）是单位时间内成功传输的数据量；Packet Loss（丢包率）反映网络拥塞或不可靠程度。三者共同构成网络性能分析的三角。
  key_distinction: |
    常识中常将"网速"简化为带宽或下载速度；这三项指标区分了延迟、容量和可靠性三个独立维度。
  why_it_matters: |
    下游 skill 必须教会学习者用这三个维度独立分析网络性能，避免用单一指标做错误的设计决策。
  tags: [networking, performance, measurement, latency]

---

## 数据系统与可靠性

- id: g12
  term: Reliability / Scalability / Maintainability (RSM)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    DDIA 开篇提出的数据系统三大核心目标：Reliability（在故障面前继续正确工作）、Scalability（负载增长时保持性能）、Maintainability（支持长期演化）。
  key_distinction: |
    常识中"好系统"往往被等同于"功能正确"或"速度快"；RSM 将讨论提升到系统在时间、负载、人员变动下的持续正确性。
  why_it_matters: |
    这是 DDIA 的元框架，也是现代数据系统设计的北极星。下游 skill 必须以此作为评估系统的首要标准。
  tags: [data-systems, reliability, scalability, maintainability]

- id: g13
  term: Consistency Spectrum (一致性谱系)
  type: term
  source_books: ["Designing Data-Intensive Applications", "Readings in Database Systems"]
  author_definition: |
    从最终一致性（Eventual Consistency）到线性一致性（Linearizability），再到可串行化（Serializability），构成一个由弱到强的连续谱。不同强度对应不同的性能、可用性和实现复杂度。
  key_distinction: |
    常识中"一致性"常被当作二元属性（一致/不一致）；而在这些书中，它是一个需要显式选择的设计参数。
  why_it_matters: |
    下游 skill 若不能澄清一致性谱系，学习者会在分布式系统设计中做出灾难性的默认假设（如假设所有系统都是强一致的）。
  tags: [distributed-systems, consistency, data-systems, tradeoff]

- id: g14
  term: Eventual Consistency (最终一致性)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    在没有新更新的情况下，副本最终会变得一致。它允许暂时的不一致以换取高可用性和低延迟。
  key_distinction: |
    常识中"最终一致"容易被误解为"差不多一致"或"不可靠"；实际上它是一种被严格定义、在特定场景下经过深思熟虑的折中。
  why_it_matters: |
    下游 skill 需要消除对最终一致性的偏见，让学习者理解在哪些业务场景下它是正确且必要的选择。
  tags: [distributed-systems, consistency, availability, tradeoff]

- id: g15
  term: Linearizability (线性一致性)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    每个操作似乎在调用的某个瞬间原子地完成，且与全局实时顺序一致。它提供了最强的单对象一致性保证，是分布式系统的"可串行化快照"。
  key_distinction: |
    常识中容易将线性一致性与"可串行化"混淆；线性一致性只保证单操作看起来原子执行，而可串行化涉及多操作事务的交错。
  why_it_matters: |
    下游 skill 必须区分线性一致性和可串行化，否则学习者会在设计并发控制时错误地套用概念。
  tags: [distributed-systems, consistency, concurrency, formalism]

- id: g16
  term: CAP Theorem (CAP 定理)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    在分布式系统中，一致性（Consistency）、可用性（Availability）、分区容错性（Partition Tolerance）三者不可兼得，最多同时满足两项。Kleppmann 进一步指出，CAP 是过于简化的营销工具，真实设计是更细粒度的权衡。
  key_distinction: |
    常识中 CAP 被误用为"必须三选二"的绝对法则；实际上分区容错性是分布式系统的现实约束，真正的权衡只在 C 和 A 之间，且是连续的。
  why_it_matters: |
    下游 skill 需要纠正对 CAP 的教条式理解，避免学习者用"我们选择了 AP"来掩盖设计上的懒惰。
  tags: [distributed-systems, consistency, availability, myth-busting]

- id: g17
  term: ACID
  type: term
  source_books: ["Readings in Database Systems", "Designing Data-Intensive Applications"]
  author_definition: |
    原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）的缩写，是事务处理系统的经典保证。不同数据库对 ACID 的实现强度差异很大。
  key_distinction: |
    常识中 ACID 被视为数据库的"标准认证"；实际上它更像是一个营销术语，各厂商对隔离级别的实现可能大相径庭。
  why_it_matters: |
    下游 skill 必须让学习者理解 ACID 不是黑箱保证，而是需要逐层拆解的具体机制（如 2PL、MVCC、WAL）。
  tags: [databases, transactions, reliability, formalism]

- id: g18
  term: Two-Phase Locking (2PL, 两阶段锁)
  type: term
  source_books: ["Readings in Database Systems"]
  author_definition: |
    事务并发控制的一种悲观策略：事务在 growing phase 获取锁，在 shrinking phase 释放锁。严格两阶段锁（Strict 2PL）直到事务结束才释放写锁，可防止级联回滚。
  key_distinction: |
    常识中容易将"锁"与操作系统中的互斥锁混为一谈；2PL 是数据库事务级别的并发控制协议，有明确的阶段划分和死锁处理机制。
  why_it_matters: |
    下游 skill 需要区分操作系统锁和数据库 2PL，否则学习者会错误地将低层同步原语直接套用到事务设计中。
  tags: [databases, concurrency, transactions, locking]

- id: g19
  term: MVCC (多版本并发控制)
  type: term
  source_books: ["Readings in Database Systems", "Designing Data-Intensive Applications"]
  author_definition: |
    通过维护数据的多个时间戳版本，使读操作无需阻塞写操作，从而实现高并发下的快照隔离。PostgreSQL 和 MySQL InnoDB 都广泛采用。
  key_distinction: |
    常识中并发控制常被等同于"加锁"；MVCC 展示了另一种思路：通过冗余（保存历史版本）来消除读写冲突。
  why_it_matters: |
    下游 skill 必须让学习者理解 MVCC 是现代 OLTP 系统的核心机制，也是"读不阻塞写"的工程实现基础。
  tags: [databases, concurrency, performance, isolation]

- id: g20
  term: Replication Lag (复制延迟)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    主从复制架构中，从副本落后于主节点的时间差。它导致用户可能读到旧数据，是最终一致性系统中最常见的异常来源。
  key_distinction: |
    常识中"复制"常被理解为即时同步；实际上复制延迟是分布式系统的常态，且可能从毫秒到分钟不等。
  why_it_matters: |
    下游 skill 必须让学习者将复制延迟视为第一阶级的设计约束，而不是可以忽略的"边缘情况"。
  tags: [distributed-systems, data-systems, consistency, reliability]

- id: g21
  term: Partitioning (分区/分片)
  type: term
  source_books: ["Designing Data-Intensive Applications", "Readings in Database Systems"]
  author_definition: |
    将大型数据集拆分为更小的子集，分布到多个节点上存储和处理。分区策略（如范围分区、哈希分区）直接影响查询效率、负载均衡和再平衡成本。
  key_distinction: |
    常识中"分库分表"常被当作性能瓶颈时的应急手段；在这些书中，分区是数据系统架构的一级设计决策，需要在数据模型阶段就考虑。
  why_it_matters: |
    下游 skill 需要教会学习者：分区不是后期的优化补丁，而是决定系统可扩展性上限的架构基石。
  tags: [distributed-systems, data-systems, scalability, architecture]

- id: g22
  term: Derived Data (派生数据)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    通过转换原始数据得到的数据，如索引、缓存、物化视图、搜索索引、推荐结果。派生数据的存在意味着系统存在多个数据表示，需要保持它们之间的一致性。
  key_distinction: |
    常识中容易将数据库视为"单一真相来源"；DDIA 指出现代系统通常由多个存储系统（OLTP、OLAP、缓存、搜索）组成，每个都持有派生数据。
  why_it_matters: |
    下游 skill 必须打破"单一数据库"的幻觉，让学习者理解数据流架构和派生数据同步是现代系统的核心挑战。
  tags: [data-systems, architecture, data-pipeline, consistency]

- id: g23
  term: OLTP / OLAP
  type: term
  source_books: ["Readings in Database Systems", "Designing Data-Intensive Applications"]
  author_definition: |
    OLTP（在线事务处理）面向大量短事务的随机读写，强调低延迟和高并发；OLAP（在线分析处理）面向复杂查询和批量扫描，强调高吞吐。两者在存储布局（行存 vs 列存）、查询模式和优化策略上截然不同。
  key_distinction: |
    常识中数据库常被当作通用工具；实际上用 OLTP 系统跑 OLAP 负载或用 OLAP 系统处理事务都是典型的反模式。
  why_it_matters: |
    下游 skill 需要让学习者理解工作负载特征决定存储引擎选择，这是数据系统架构设计的第一步。
  tags: [databases, data-systems, architecture, workload]

- id: g24
  term: Data Independence (数据独立性)
  type: term
  source_books: ["Readings in Database Systems"]
  author_definition: |
    将数据的逻辑组织（逻辑独立性）和物理存储（物理独立性）与应用程序分离。关系模型和查询优化器的出现正是为了实现这一抽象。
  key_distinction: |
    常识中"数据独立性"可能被视为数据库的一个特性；在 Red Book 中，它是数据库系统演进的核心理论动力，是软件工程中"关注点分离"在数据领域的具体化。
  why_it_matters: |
    下游 skill 必须传递数据独立性作为架构设计的元原则：接口与实现的分离是系统可演化的根本。
  tags: [databases, abstraction, architecture, evolution]

---

## 并发与操作系统

- id: g25
  term: Concurrency (并发)
  type: term
  source_books: ["Operating Systems: Three Easy Pieces", "Computer Architecture", "Designing Data-Intensive Applications"]
  author_definition: |
    OSTEP 中，并发指多个执行流在重叠的时间段内推进，它们可能通过多线程、多进程或中断实现。并发引入了全新的故障模式：竞态条件、死锁、非确定性。
  key_distinction: |
    常识中常将并发与并行混用；并发是"结构"（多个任务同时存在），并行是"执行"（多个任务同时运行）。并发可以在单核上发生，而并行需要多核。
  why_it_matters: |
    下游 skill 若不能区分并发与并行，学习者会错误地认为"没有多核就不需要学并发"，从而忽视单核上的竞态条件和同步问题。
  tags: [concurrency, os, distributed-systems, correctness]

- id: g26
  term: Context Switch (上下文切换)
  type: term
  source_books: ["Operating Systems: Three Easy Pieces"]
  author_definition: |
    操作系统保存当前进程状态并恢复另一个进程状态的过程。它是实现 CPU 虚拟化和多道程序设计的核心机制，但本身也有开销（寄存器保存、缓存失效、TLB 刷新）。
  key_distinction: |
    常识中上下文切换常被简化为"切换进程"；实际上它是一系列精确的状态保存/恢复操作，其开销直接影响系统吞吐量和响应延迟。
  why_it_matters: |
    下游 skill 需要让学习者理解上下文切换不是免费的，高并发设计必须在并发度和切换开销之间取得平衡。
  tags: [os, performance, virtualization, scheduling]

- id: g27
  term: Page Table / TLB
  type: term
  source_books: ["Operating Systems: Three Easy Pieces", "Computer Architecture"]
  author_definition: |
    页表是操作系统维护的虚拟地址到物理地址映射数据结构；TLB（Translation Lookaside Buffer）是 CPU 中的高速缓存，用于加速地址翻译。
  key_distinction: |
    常识中内存访问被简化为"直接读写地址"；实际上每次内存访问都可能涉及页表遍历和 TLB 命中/未命中，这是理解内存性能的关键。
  why_it_matters: |
    下游 skill 必须揭示"内存访问"背后的翻译机制，否则学习者无法理解为什么大页（huge page）和 TLB 优化能带来数量级性能提升。
  tags: [os, memory, performance, hardware]

- id: g28
  term: Lock / Condition Variable
  type: term
  source_books: ["Operating Systems: Three Easy Pieces"]
  author_definition: |
    锁（Lock/Mutex）用于互斥访问共享资源；条件变量（Condition Variable）用于线程间的事件通知和等待。两者是构建更复杂同步原语的基础。
  key_distinction: |
    常识中"锁"常被过度使用或误用（如用锁保护过多代码）；条件变量则常被初学者忽视，导致忙等或错误实现生产者-消费者模式。
  why_it_matters: |
    下游 skill 需要教会学习者：锁解决互斥，条件变量解决同步，两者缺一不可。错误使用是并发 Bug 的主要来源。
  tags: [concurrency, os, synchronization, correctness]

- id: g29
  term: Deadlock (死锁)
  type: term
  source_books: ["Operating Systems: Three Easy Pieces", "Readings in Database Systems"]
  author_definition: |
    多个进程/线程因循环等待资源而永久阻塞的状态。四个必要条件：互斥、持有并等待、不可抢占、循环等待。预防、避免、检测和恢复是四种处理策略。
  key_distinction: |
    常识中死锁常被等同于"程序卡死"；实际上死锁是资源分配理论中的形式化概念，有明确的诊断和解决框架。
  why_it_matters: |
    下游 skill 必须将死锁从"神秘 Bug"转化为可分析、可预防的系统属性，这是并发系统设计者必备的思维工具。
  tags: [concurrency, os, databases, reliability]

- id: g30
  term: Journaling / WAL (预写日志)
  type: term
  source_books: ["Operating Systems: Three Easy Pieces", "Readings in Database Systems"]
  author_definition: |
    在修改磁盘数据结构之前，先将变更记录到日志中。崩溃后可通过日志重放恢复一致性。文件系统（如 ext3）和数据库（如 InnoDB）都广泛采用。
  key_distinction: |
    常识中日志常被理解为"操作记录"或"审计追踪"；在持久化系统中，日志是崩溃恢复的核心机制，是正确性的保障而非可选功能。
  why_it_matters: |
    下游 skill 需要让学习者理解 WAL 是持久化系统的基石：没有日志，就没有可靠的崩溃恢复。
  tags: [os, databases, persistence, reliability]

---

## 程序设计与抽象

- id: g31
  term: Abstraction (抽象)
  type: term
  source_books: ["SICP", "Operating Systems: Three Easy Pieces", "Computer Networking", "Readings in Database Systems"]
  author_definition: |
    SICP 中，抽象是控制复杂性的首要工具，包括过程抽象（隐藏计算过程）、数据抽象（隐藏数据表示）和元语言抽象（创建新语言）。这一思想贯穿所有书籍：网络的分层、OS 的虚拟化、数据库的数据独立性都是抽象的不同形态。
  key_distinction: |
    常识中"抽象"常被贬义化为"不具体"或"脱离实际"；在 CS 中，抽象是精确的边界定义，是工程可控性的来源。
  why_it_matters: |
    抽象是 8 本书共同的第一性原理。下游 skill 若不能澄清抽象的工程意义，学习者会陷入"要么太高层、要么太底层"的两极。
  tags: [abstraction, first-principles, design-pattern, sicp]

- id: g32
  term: Higher-Order Procedure (高阶过程)
  type: term
  source_books: ["SICP"]
  author_definition: |
    以过程为参数或返回值的过程。它允许将控制结构（如 map、filter、reduce）本身参数化，是函数式编程和许多现代语言（如 Python、JavaScript）的核心机制。
  key_distinction: |
    常识中"函数"常被理解为执行特定任务的代码块；高阶过程将"过程"提升为一等公民，改变了程序组织的方式。
  why_it_matters: |
    下游 skill 需要让学习者理解高阶过程不仅是语法糖，更是一种强大的设计模式：将变化的部分参数化，将不变的部分固化。
  tags: [sicp, abstraction, functional-programming, design-pattern]

- id: g33
  term: Environment Model (环境模型)
  type: term
  source_books: ["SICP"]
  author_definition: |
    解释程序执行时变量绑定和作用域的形式化模型。每个过程调用创建一个新的环境帧，变量在该帧中绑定。它是理解闭包、递归和状态共享的基础。
  key_distinction: |
    常识中变量常被理解为"存储值的盒子"；环境模型将变量视为环境中的绑定，强调作用域和生命周期。
  why_it_matters: |
    下游 skill 必须引入环境模型，否则学习者会在闭包、this 绑定、变量提升等概念上产生持久性误解。
  tags: [sicp, semantics, programming-languages, fundamentals]

- id: g34
  term: Data-Directed Programming (数据导向编程)
  type: term
  source_books: ["SICP"]
  author_definition: |
    将操作的选择从固定的控制流（如 cond/switch）转移到数据本身，通过查表或消息分发决定调用哪个过程。它是面向对象多态和插件架构的理论前身。
  key_distinction: |
    常识中程序控制流常被视为核心逻辑；数据导向编程将"操作选择"本身数据化，使系统更易于扩展。
  why_it_matters: |
    下游 skill 需要让学习者看到现代框架（如 React 的虚拟 DOM diff、RPC 分发器）与 SICP 中 dispatch table 的深层联系。
  tags: [sicp, abstraction, design-pattern, polymorphism]

- id: g35
  term: Lazy Evaluation (惰性求值)
  type: term
  source_books: ["SICP"]
  author_definition: |
    延迟表达式的求值直到其值真正被需要。SICP 中用惰性求值构建无限流（Stream），实现了按需计算和内存效率的统一。
  key_distinction: |
    常识中程序执行被理解为"按顺序计算"；惰性求值将计算与消费解耦，是理解生成器、迭代器、Promise 等现代机制的钥匙。
  why_it_matters: |
    下游 skill 需要澄清：惰性求值不仅是性能优化，更是一种控制计算边界的抽象工具。
  tags: [sicp, evaluation-strategy, streams, performance]

---

## 算法与数学

- id: g36
  term: Structural Induction (结构归纳法)
  type: term
  source_books: ["Mathematics for Computer Science"]
  author_definition: |
    对递归定义的数据结构（如树、列表）进行归纳证明的方法。基例对应原子元素，归纳步假设子结构成立并证明整体成立。
  key_distinction: |
    常识中"归纳法"常指数学归纳法（对自然数）；结构归纳法是其在计算机科学中的自然延伸，直接对应递归算法和数据结构的正确性证明。
  why_it_matters: |
    下游 skill 必须将结构归纳法与递归算法绑定教学，让学习者具备形式化验证算法正确性的能力。
  tags: [mathematics, proof, recursion, correctness]

- id: g37
  term: Invariant (不变量)
  type: term
  source_books: ["Mathematics for Computer Science", "SICP", "Advanced Algorithms and Data Structures"]
  author_definition: |
    在算法执行过程中始终保持为真的命题。循环不变量用于证明迭代算法的正确性；数据结构不变量用于保证抽象数据类型的行为一致性。
  key_distinction: |
    常识中"不变量"可能被视为静态属性；实际上它是动态执行过程中的稳定锚点，是理解和调试算法的核心工具。
  why_it_matters: |
    不变量是算法正确性证明和并发程序设计的通用语言。下游 skill 必须将其作为分析算法的首要 lens。
  tags: [mathematics, proof, algorithms, correctness]

- id: g38
  term: Probabilistic Data Structure (概率数据结构)
  type: term
  source_books: ["Advanced Algorithms and Data Structures", "Designing Data-Intensive Applications"]
  author_definition: |
    通过允许微小的错误概率来换取巨大的空间或时间效率的数据结构。Bloom Filter 是典型代表：以可能误判（false positive）为代价，实现常数时间的成员查询和极低的内存占用。
  key_distinction: |
    常识中数据结构常被默认为"精确"的；概率数据结构挑战了这一假设，将"确定性"本身作为可权衡的设计参数。
  why_it_matters: |
    下游 skill 需要让学习者理解：在大规模系统中，"足够好"的概率保证往往比"绝对正确"的精确实现更有工程价值。
  tags: [algorithms, data-structures, probability, scalability]

- id: g39
  term: Bloom Filter
  type: term
  source_books: ["Advanced Algorithms and Data Structures", "Designing Data-Intensive Applications"]
  author_definition: |
    一种空间高效的概率型集合成员查询数据结构。查询结果可能是"可能在集合中"（有假阳性）或"一定不在集合中"（无假阴性）。
  key_distinction: |
    常识中可能将 Bloom Filter 视为普通哈希表的替代品；实际上它是一种在特定约束下（空间极度受限、允许少量误判）的专用工具。
  why_it_matters: |
    Bloom Filter 是理解"用概率换效率"设计哲学的最佳案例。下游 skill 需要明确其适用边界，避免学习者将其当作通用数据结构滥用。
  tags: [algorithms, data-structures, probability, systems]

- id: g40
  term: Approximation Algorithm (近似算法)
  type: term
  source_books: ["Advanced Algorithms and Data Structures", "Mathematics for Computer Science"]
  author_definition: |
    对于 NP-hard 问题，在多项式时间内给出接近最优解的算法。近似比（approximation ratio）用于量化解的质量保证。
  key_distinction: |
    常识中算法常被默认为"求精确最优解"；近似算法承认计算不可行性，将目标从"最优"转向"可证明地接近最优"。
  why_it_matters: |
    下游 skill 必须打破"算法必须精确"的执念。真实世界中的调度、路由、聚类等问题大多使用近似算法。
  tags: [algorithms, complexity, optimization, tradeoff]

- id: g41
  term: Expectation Linearity (期望线性性)
  type: term
  source_books: ["Mathematics for Computer Science"]
  author_definition: |
    随机变量之和的期望等于期望之和，无论变量是否独立。这一性质极大地简化了复杂概率问题的分析。
  key_distinction: |
    常识中概率分析常被视为需要复杂的联合分布计算；期望线性性提供了一个无需独立假设的强大工具。
  why_it_matters: |
    下游 skill 需要让学习者掌握期望线性性，这是分析随机算法和哈希表性能的核心技巧。
  tags: [mathematics, probability, algorithms, analysis]

---

## 元设计概念

- id: g42
  term: Top-Down Approach (自顶向下方法)
  type: term
  source_books: ["Computer Networking"]
  author_definition: |
    Kurose & Ross 倡导的网络学习方法：从应用层的需求和问题出发，逐层向下探索底层机制。强调"先理解为什么，再理解怎么做"。
  key_distinction: |
    常识中技术学习常采用自底向上（从物理层到应用层）；自顶向下将动机置于机制之前，更符合工程师的问题解决思维。
  why_it_matters: |
    下游 skill 需要传递这种学习方法论：在复杂系统中，从用户问题和高层目标出发，再向下拆解实现，是更高效的理解路径。
  tags: [pedagogy, networking, methodology, systems-thinking]

- id: g43
  term: Consensus (共识)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    分布式系统中多个节点就某一值达成一致的过程。Paxos、Raft 等算法在部分故障和网络分区下仍能保证安全性。
  key_distinction: |
    常识中"共识"可能被理解为简单的投票或多数决；分布式共识涉及严格的容错边界、活性和安全性权衡，是理论上深刻、工程上困难的课题。
  why_it_matters: |
    下游 skill 必须让学习者敬畏共识问题：许多分布式系统故障的根源都是对共识算法的误用或过度简化。
  tags: [distributed-systems, consensus, reliability, theory]

- id: g44
  term: Stream Processing (流处理)
  type: term
  source_books: ["Designing Data-Intensive Applications"]
  author_definition: |
    以事件流为基本数据模型，对数据进行持续、低延迟的处理。与批处理相对，流处理强调数据的时效性和增量计算。
  key_distinction: |
    常识中数据处理常被二分为"批处理"和"实时处理"；DDIA 指出两者之间的界限正在模糊（如 Lambda 架构、Kappa 架构），流处理也可以实现正确性保证。
  why_it_matters: |
    下游 skill 需要打破"批处理=准确，流处理=近似"的刻板印象，让学习者理解现代流处理系统（如 Flink）也能提供强一致性。
  tags: [data-systems, streaming, architecture, scalability]

- id: g45
  term: RAID
  type: term
  source_books: ["Operating Systems: Three Easy Pieces"]
  author_definition: |
    冗余磁盘阵列，通过将数据条带化和镜像/校验分布在多个磁盘上，提升性能、容量或可靠性。不同级别（RAID 0/1/4/5/6）对应不同的冗余和性能权衡。
  key_distinction: |
    常识中 RAID 常被等同于"数据备份"；实际上它是一种在磁盘层面实现的冗余和并行 I/O 技术，不能替代离线备份。
  why_it_matters: |
    下游 skill 需要纠正对 RAID 的误解，避免学习者将其作为灾难恢复的唯一手段。
  tags: [os, storage, reliability, performance]

---

## 统计

- **总词条数**: 45
- **覆盖书籍**: 8/8
- **跨书术语**: Virtualization, Concurrency, Abstraction, Consistency, Locality, Performance, Reliability 等
- **下游 skill 重点**: 抽象边界设计、定量性能分析、并发与容错、数据系统可靠性评估
