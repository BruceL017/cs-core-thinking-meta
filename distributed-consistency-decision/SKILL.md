---
name: distributed-consistency-decision
description: |
  用户在设计分布式数据库、缓存、消息队列或跨服务数据系统时，需要在一致性、可用性、延迟之间做权衡；或在讨论 CAP/PACELC、Linearizability、Serializability、2PC/Saga、向量时钟、Raft/Paxos 等话题时寻求决策框架。
  不适用于：纯概念查询（如"什么是 CAP 定理"）、单节点数据库调优、不涉及一致性级别的简单缓存选型。
source_book: 《Designing Data-Intensive Applications》Martin Kleppmann
source_chapter: "Chapter 7 & 9 — Consistency and Consensus"
tags: [distributed-systems, consistency, CAP, PACELC, linearizability, consensus]
related_skills: []
---

# 分布式一致性设计的 CAP/PACELC 决策矩阵

## R — 原文 (Reading)

> "There is some similarity between distributed consistency models and the hierarchy of transaction isolation levels we discussed previously. But while there is some overlap, they are mostly independent concerns: transaction isolation is primarily about avoiding race conditions due to concurrently executing transactions, whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults."
>
> — Martin Kleppmann, DDIA Chapter 7 & 9

---

## I — 方法论骨架 (Interpretation)

这个框架教你如何在分布式系统中为数据副本选择正确的一致性级别。核心逻辑分三层：

1. **先区分问题域**：事务隔离（Isolation）解决的是并发事务之间的竞态条件；分布式一致性（Consistency）解决的是多副本在延迟和故障下的状态协调。两者相关但独立。

2. **用 PACELC 做双模决策**：
   - 分区时（P）：选 CP（牺牲可用性保一致性）还是 AP（牺牲强一致性保可用性）。
   - 无分区时（EL）：选降低延迟（异步复制）还是增强一致性（同步复制）。

3. **将一致性模型映射到实现机制**：
   - 线性一致性（Linearizability）→ 单主复制、共识协议（Paxos/Raft）、全局锁；
   - 顺序一致性 → 分布式日志（Kafka、WAL）；
   - 因果一致性 → 向量时钟、版本向量、Lamport 时间戳；
   - 最终一致性 → 多主/无主复制、读修复、反熵。

4. **关键操作必须硬约束**：分布式锁、主节点选举、唯一性约束、fencing token 等操作必须依赖线性一致性或共识，不可用最终一致性凑合。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: Amazon Dynamo 的 AP 选择与向量时钟冲突检测
- **问题**: 电商购物车服务需要在网络分区时仍然可写，不能因节点失联而拒绝用户添加商品。
- **方法论的使用**: 采用 CAP 中的 AP 路径：分区时优先保证可用性，接受多主副本上的并发写入冲突；使用向量时钟（vector clocks）检测并发更新，将冲突解决推迟到读取阶段由应用层处理。
- **结论**: 购物车场景天然可接受短暂不一致，但绝不能接受写入失败。
- **结果**: Dynamo 成为高可用键值存储的标杆，验证了"一致性是设计选择而非系统默认属性"。

### 案例 2: Twitter 时间线的混合一致性策略
- **问题**: 明星用户发推时，fan-out-on-write 的写入量爆炸；普通用户读取时却要求低延迟。
- **方法论的使用**: 对普通用户采用 fan-out-on-write（预计算时间线，接受最终一致性）；对明星用户采用 fan-out-on-read（读取时实时合并，降低写入峰值）。
- **结论**: 同一系统内不同数据子集可以采用不同的一致性策略，取决于读写比例和延迟要求。
- **结果**: 混合架构在扩展性和延迟之间取得了量化平衡。

### 案例 3: LinkedIn 个人资料的多层存储与最终一致性
- **问题**: 单一数据库无法满足点查询、全文搜索、缓存热数据等多种访问模式。
- **方法论的使用**: 为每种访问模式选择专门的存储层（Voldemort 缓存、ESPI 搜索索引、Oracle 主库），通过 Databus 变更日志保持层间最终一致。
- **结论**: Polyglot persistence 的代价是必须在层间显式管理一致性，不能假设所有层自动同步。
- **结果**: 复杂查询性能提升，但需要接受跨层短暂不一致。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **设计跨地域多活数据库或缓存架构时**：需要在同步复制（延迟高、一致性强）和异步复制（延迟低、可能读到旧数据）之间做选择。
2. **评估是否引入分布式事务（2PC/Saga）时**：用户在纠结跨服务操作要不要用 2PC，或者担心 Saga 的补偿语义不够安全。
3. **实现分布式锁、唯一 ID 生成、主节点选举时**：这些操作对一致性要求极高，必须明确使用共识协议或线性一致性存储。
4. **系统出现脑裂、数据冲突或读取到旧状态后**：需要回溯一致性设计决策，找出当初 CAP/PACELC 权衡的盲点。

### 语言信号 (用户的话里出现这些就应激活)

- "我们这套系统该选 CP 还是 AP？"
- "这个场景需不需要线性一致性？"
- "2PC 和 Saga 哪个更适合跨服务事务？"
- "能不能用最终一致性来做分布式锁？"
- "读写复制延迟导致用户看到旧数据，要不要改成同步复制？"
- "Raft 和 Paxos 在这个场景下怎么选？"

### 与相邻 skill 的区分

- 与 `concurrency-control-decision`（并发控制悲观/乐观策略选择）的区别：后者聚焦单库事务隔离级别（Serializable vs Snapshot Isolation），本 skill 聚焦多副本间的分布式一致性（Linearizability vs Eventual Consistency）。
- 与 `quantitative-performance-framework` 的区别：后者关注负载建模与性能测量，本 skill 在量化负载的基础上专门解决一致性-可用性-延迟的三角权衡。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后，agent 应按以下步骤执行:

1. **澄清业务场景与故障模型**
   - 完成标准: 明确以下四个问题的答案：
     a) 数据是读多写少还是读写均衡？
     b) 延迟要求是多少（毫秒级 vs 秒级可接受）？
     c) 网络分区发生时，业务更容忍不可用还是更容忍读到旧数据？
     d) 是否存在必须全局唯一的操作（锁、选主、唯一约束）？
   - 判停条件: 若用户无法回答上述问题，则引导用户用具体数字和场景描述，而非跳到结论。

2. **应用 PACELC 决策矩阵**
   - 完成标准: 给出明确的 PACELC 定位：
     - 分区时（P）：CP 或 AP；
     - 无分区时（EL）：优先 Latency 还是 Consistency。
   - 判停条件: 若用户场景涉及金融交易、库存扣减、唯一性约束，则强制定位到 CP+Consistency（EL 中的 C），并警告不可用 AP 或最终一致性。

3. **映射到具体一致性模型与实现机制**
   - 完成标准: 将 PACELC 结论映射到以下至少一项：
     - 线性一致性 → Raft/Paxos/单主同步复制/ZooKeeper/etcd；
     - 顺序一致性 → 分布式日志/Kafka/WAL；
     - 因果一致性 → 向量时钟/Lamport 时间戳/版本向量（Dynamo/Voldemort 风格）；
     - 最终一致性 → 多主/无主复制 + 读修复 + 反熵（Cassandra/Dynamo 风格）。
   - 判停条件: 若步骤 2 判定为 CP+Consistency，但用户提议用无主复制或异步复制，则明确否决并解释风险。

4. **给出反模式警告与验证建议**
   - 完成标准:
     a) 引用至少一个相关 counter-example（ce08–ce11）说明常见失败模式；
     b) 建议显式验证手段：混沌测试网络分区、监控复制滞后（replication lag）、定期审计多副本数据一致性。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 纯概念解释请求（如"解释一下 CAP 定理"），没有具体的系统设计或决策场景。
- 单节点数据库的索引优化或查询调优，不涉及多副本一致性。
- 用户只是想要一个具体的技术选型清单（"Redis 和 Memcached 哪个好"），而没有涉及一致性级别、分区容错或复制策略的权衡。

### 作者在书中警告的失败模式

- **ce08 — 忽略网络分区（假设网络可靠）**: 若系统未显式定义分区时的行为策略，未设置 RPC 超时与熔断，分区发生时将出现数据不一致、服务雪崩或脑裂。CAP 定理的核心价值正是强迫设计者提前做出选择，而不是假装分区不会发生。
- **ce09 — 过早优化一致性而牺牲可用性**: 在社交点赞、评论数、商品库存预估等可接受最终一致性的场景强制使用线性一致性或分布式事务，会导致读延迟与跨地域 RTT 强相关，且轻微网络抖动即大量失败。强一致性很贵，不要强加给不需要它的业务。
- **ce10 — 将分布式事务当作本地事务处理**: 2PC 的协调者是单点故障，崩溃后参与者必须阻塞等待；跨服务事务的延迟比本地事务高数个数量级。若业务可接受 Saga 的补偿语义，应优先避免 2PC。
- **ce11 — 信任单一数据源而不验证**: 在多层存储（缓存+搜索索引+主库）架构中，若下游直接消费上游数据而不做 schema 验证、checksum 或异常检测，复制滞后或写入失败会导致静默数据腐败。最终一致性系统必须配套监控与审计。

### 作者的盲点 / 时代局限

- Kleppmann 的写作背景主要是 2017 年及之前的西方互联网基础设施，对于 Serverless 架构、边缘计算、低带宽环境下的 CAP 权衡讨论较少。
- 对 LLM 时代的向量数据库、实时推理系统中的一致性需求（如模型参数同步、KV cache 一致性）覆盖不足。
- 书中假设故障主要是崩溃停止（crash-stop）或网络分区，对拜占庭故障（Byzantine faults）和恶意攻击场景的讨论有限。

### 容易混淆的邻近方法论

- **Serializability vs Linearizability**: Serializability 是事务隔离的黄金标准（保证事务执行等价于某个串行顺序），Linearizability 是分布式一致性的黄金标准（保证每个操作看起来在调用的某个瞬间原子完成）。一个系统可以 Serializable 但不 Linearizable，反之亦然。
- **2PC vs Consensus**: 2PC 是原子提交协议，解决的是"所有节点都提交或都回滚"；Paxos/Raft 是共识协议，解决的是"多个节点对一个值达成一致"。2PC 的协调者是单点，共识协议通过选举机制避免单点。
- **Vector Clock vs Lamport Timestamp**: Vector Clock 可以检测并发冲突（知道 A 发生在 B 之前、B 发生在 A 之前、还是 A 与 B 并发）；Lamport Timestamp 只能建立偏序，无法区分并发与因果关系。

---

## 相关 skills

- depends-on: quantitative-performance-analysis, layered-end-to-end-decision
- contrasts-with: concurrency-strategy-selection, crash-recovery-persistence-design
- composes-with: data-system-reliability-assessment, safety-liveness-properties


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 100% (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
