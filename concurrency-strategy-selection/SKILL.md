---
name: concurrency-strategy-selection
description: |
  用户在数据库事务、并发数据结构或分布式锁的设计中面临策略选型时激活。
  典型信号：在悲观锁/乐观锁/MVCC/2PL/SSI/OCC之间犹豫；提到读写比例、冲突概率、事务长度或隔离级别。
  不适用于：纯概念定义查询、单线程代码实现、与并发控制无关的分布式一致性讨论。
source_book: 《Operating Systems: Three Easy Pieces》/《Readings in Database Systems》/《Designing Data-Intensive Applications》
source_chapter: OSTEP Ch.28–31 / RedBook Ch.3 / DDIA Ch.7
tags: [concurrency-control, pessimistic-locking, optimistic-concurrency, MVCC, 2PL, SSI, OCC]
related_skills: []
---

# 并发控制：悲观 vs 乐观策略选择框架

## R — 原文 (Reading)

> "Our first paper on transactions, from Gray et al., introduces two classic ideas: multi-granularity locking and multiple lock modes." / "Serializable snapshot isolation (SSI) is not linearizable: by design, it makes reads from a consistent snapshot, to avoid lock contention between readers and writers."
>
> — Gray et al. / Kleppmann, OSTEP Ch.28–31 / RedBook Ch.3 / DDIA Ch.7

---

## I — 方法论骨架 (Interpretation)

并发控制策略的选择本质上是在"正确性成本"与"性能成本"之间做量化权衡。
悲观策略（锁、两阶段锁 2PL）假设冲突频繁，通过预先获取锁来阻止并发事务的干扰，代价是锁等待、死锁风险和吞吐量上限。
乐观策略（乐观并发控制 OCC、多版本并发控制 MVCC、串行化快照隔离 SSI）假设冲突稀少，允许事务自由执行，在提交时检测冲突并回滚，代价是验证开销、版本存储和回滚成本。
决策时必须显式量化四个维度：写冲突概率、读写比例、事务长度、以及网络/内存延迟。
冲突高、事务长、网络延迟大时，悲观策略的锁持有时间会被放大，反而可能劣于乐观策略；而关键财务操作或对账任务，即使冲突低，也可能需要严格的串行化保证。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: xv6 的进程调度与定时器中断
- **问题**: 操作系统如何让多个进程安全地共享同一颗 CPU，同时防止任一进程独占处理器？
- **方法论的使用**: xv6 采用了一种最简化的"悲观"资源控制策略——轮询调度配合定时器中断。操作系统通过硬件中断强制剥夺 CPU 使用权，保存/恢复寄存器状态，实现严格的时分复用。这相当于用"强制锁"（时间片）来保证任何进程都不会与其他进程在物理 CPU 上真正并发执行。
- **结论**: 当共享资源（CPU）极度稀缺且冲突概率为 100% 时，预先分配、强制切换的悲观策略是最可靠的选择。
- **结果**: xv6 以数百行代码实现了可运行的多任务系统，证明了在底层硬件上，"性能（直接执行）"与"控制（中断、特权级切换）"必须显式权衡。

### 案例 2: Linux CFS 的红黑树设计
- **问题**: 如何在多核系统上公平地调度数千个线程，同时避免固定时间片带来的僵化？
- **方法论的使用**: CFS 将调度策略抽象为"最小化虚拟运行时间（vruntime）"，而实现机制则委托给红黑树。这是一种从"强制悲观锁"（固定时间片）向"自适应统计策略"的演进：不再一刀切地剥夺 CPU，而是动态找出最"亏欠"的任务执行。
- **结论**: 并发控制中的策略抽象（公平份额）与实现机制（数据结构）可以分离；好的抽象需要匹配高效的底层机制才能落地。
- **结果**: Linux 调度器在 O(log n) 时间内完成插入和删除，支持从嵌入式设备到数千核服务器的扩展，展示了策略-机制分离在并发资源管理中的威力。

---

## A2 — 触发场景 (Future Trigger)

### 用户会在什么情境下需要这个 skill?

1. **数据库事务隔离级别选型**：团队在高并发 OLTP 系统中争论该用 InnoDB 行锁、Snapshot Isolation 还是 Serializable Snapshot Isolation（SSI）。
2. **分布式锁 vs 版本向量冲突检测**：跨数据中心的写操作延迟高，分布式锁导致 P99 激增，考虑换成基于版本向量的乐观冲突检测。
3. **并发数据结构的锁粒度设计**：实现 B+树、跳表或哈希表时，在读写锁、无锁结构（lock-free）、RCU 或 OCC 之间做选择。

### 语言信号 (用户的话里出现这些就应激活)

- "我们该用悲观锁还是乐观锁？"
- "这个场景下 MVCC 和两阶段锁哪个更合适？"
- "高并发下事务总是回滚/死锁，是不是该换 SSI 或 OCC？"
- "读写比例 9:1，是不是应该用快照隔离而不是 Serializable？"
- "分布式锁延迟太高了，能不能改成基于版本的冲突检测？"

### 与相邻 skill 的区分

- 与 `distributed-consistency-cap-pacelc` 的区别: 本 skill 聚焦于**事务内部**的并发控制策略（锁 vs 多版本 vs 乐观验证），而非**副本之间**的一致性模型（线性一致性、顺序一致性、最终一致性）。
- 与 `scheduling-workload-policy-framework` 的区别: 本 skill 关注共享数据的**正确性保证**（隔离级别、竞态条件、串行化），而非 CPU/任务调度的**响应时间与公平性**。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后，agent 应按以下步骤执行:

1. **量化工作负载特征**
   - 完成标准: 明确给出读写比例、写冲突概率（热点行/记录比例）、事务平均长度（操作数或持续时间）、以及访问模式（随机点查 vs 范围扫描）。
   - 判停条件: 若用户无法提供任何量化信息，则跳到步骤 4，给出基于典型场景的推荐区间。

2. **评估悲观策略的成本与适用性**
   - 完成标准: 计算或估算锁等待时间、死锁概率、锁升级风险（行锁→表锁），以及上下文切换/缓存失效对吞吐量的影响。
   - 判停条件: 若事务长度超过平均网络 RTT 的 3 倍，或热点行竞争率 >20%，则标记悲观策略为"高风险"。

3. **评估乐观策略的成本与适用性**
   - 完成标准: 计算或估算验证失败率、回滚代价（重试次数、业务补偿复杂度）、版本存储开销（MVCC 的 undo log/多版本保留），以及只读事务的比例收益。
   - 判停条件: 若回滚率 >10% 或事务涉及外部 I/O（无法重试），则标记乐观策略为"高风险"。

4. **输出策略决策矩阵与具体实现建议**
   - 完成标准: 用表格或清单形式给出推荐策略（如：读多写少+低冲突 → MVCC/Snapshot Isolation；高冲突+短事务 → 2PL/行锁；高冲突+长事务 → OCC 或实际串行执行；跨地域+高延迟 → SSI 或版本向量），并附带锁粒度建议（行锁 vs 页锁 vs 表锁）。

5. **明确边界条件与回退策略**
   - 完成标准: 指出在什么情况下必须切换策略（如大促期间冲突率飙升时从 OCC 回退到 2PL），以及如何预防死锁（全局锁顺序、超时、优先级继承）。

---

## B — 边界 (Boundary)

### 不要在以下情况使用此 skill

- **纯单线程应用或完全无共享状态的场景**: 如果代码只在单线程执行，或使用纯函数式不可变数据结构，不存在并发冲突，讨论锁或 MVCC 是无意义的。
- **用户仅询问概念定义而非做选型决策**: 例如"请解释什么是死锁"或"什么是 SSI"，这类问题不需要启动策略选择框架，直接给出定义即可。
- **与并发控制无关的分布式系统问题**: 例如讨论数据分片策略、负载均衡算法或副本放置拓扑，这些属于其他 skill 的范畴。

### 作者在书中警告的失败模式

- **ce12 — 错误使用信号量导致死锁**: OSTEP 作者指出，低级同步原语（如信号量）不提供死锁预防机制。若多个线程以不同顺序获取多个锁，就会形成循环等待。这警示我们：在采用悲观锁策略时，必须显式定义全局或局部的锁获取顺序（total ordering），否则策略再正确也会因实现细节而挂起。
- **ce14 — 认为更多线程总是更好**: OSTEP 作者警告，线程数超过物理核心数后，上下文切换和缓存失效会主导性能，形成"并发反扩展"。这提示我们：即使选择了理论上最优的并发控制策略，也必须将并发度与硬件能力匹配，避免锁竞争被不必要的调度开销放大。
- **ce19 — 低估并发控制的开销**: Red Book 作者明确指出，"Concurrency control is not free. Two-phase locking, in particular, can serialize transactions and limit throughput." 在高并发场景下盲目使用 Serializable 或粗粒度锁，会导致事务串行化、死锁频发，系统吞吐量远低于硬件能力。这强调了根据实际 workload 特征选择隔离级别和锁粒度的必要性。

### 作者的盲点 / 时代局限

- 这些教材成书于 2017–2021 年，对现代云原生数据库（如 Aurora、Spanner、TiDB、CockroachDB）中基于 Raft 的分布式事务优化、以及硬件事务内存（HTM）的讨论不够深入。
- 对 Serverless 场景下瞬时高并发（cold start + 突发流量）与锁策略的适配关系缺乏系统论述。

### 容易混淆的邻近方法论

- 容易与"CAP/PACELC 决策矩阵"混淆：CAP 解决的是**分区时一致性 vs 可用性**的权衡，而本 skill 解决的是**同一数据被并发访问时锁 vs 多版本 vs 乐观验证**的权衡。两者可以组合使用，但问题域不同。

---

## 相关 skills

- depends-on: quantitative-performance-analysis
- contrasts-with: distributed-consistency-decision
- composes-with: data-system-reliability-assessment, safety-liveness-properties


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 100% (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
