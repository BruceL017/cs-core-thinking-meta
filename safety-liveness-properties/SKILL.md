---
name: safety-liveness-properties
description: |
  当用户需要分析分布式系统、并发协议或容错算法的正确性属性，并在安全属性（Safety）与活性属性（Liveness）之间进行分类、验证方法选择和设计权衡时激活。
  典型信号：讨论"系统会不会丢数据/脑裂/双花"、"服务最终能否恢复/达成共识/返回结果"、"该选强一致性还是最终一致性"、"超时与重试机制如何设计"。
  不适用于：纯编程语法问题、单一节点的本地算法优化、不涉及正确性分类的通用性能调优。
source_book: 《Designing Data-Intensive Applications》Martin Kleppmann
source_chapter: "Chapter 8 — System Model and Reality"
tags: [safety, liveness, distributed-algorithms, correctness, fault-tolerance, formal-verification]
related_skills: []
---

# 安全属性与活性属性的系统正确性分类框架

## R — 原文 (Reading)

> "To clarify the situation, it is worth distinguishing between two different kinds of properties: safety and liveness properties... Safety is often informally defined as nothing bad happens, and liveness as something good eventually happens."
>
> — Martin Kleppmann, DDIA Chapter 8

---

## I — 方法论骨架 (Interpretation)

该框架将分布式与并发系统的正确性需求拆分为两个正交维度：

1. **Safety（安全属性）**：断言"坏事永远不会发生"。它是一种不变量（invariant），一旦违反就永久失效，且通常可被反例一次性证伪。典型实例包括：唯一性约束（uniqueness）、单调序列（monotonic sequence）、不可双花（no double-spending）、线性一致性的读约束（read-your-writes）、互斥锁的互斥性（mutual exclusion）。Safety 的验证通常依赖不变量推导、归纳证明或模型检查（model checking）。

2. **Liveness（活性属性）**：断言"好事最终会发生"。它描述系统的进展性（progress），在有限时间内可能暂时不成立，但在合理假设下必须最终达成。典型实例包括：可用性（availability）、最终一致性（eventual consistency）、共识达成（consensus termination）、请求必有响应（request-response）、死锁恢复。Liveness 的证明常依赖良基关系（well-founded relation）、公平调度假设（fairness）以及超时/重试/心跳机制的设计。

3. **验证方法的分野**：Safety 属性可用状态空间搜索（如 TLA+、SPIN、Paxos 的 inductive invariant）进行穷举或归纳验证；Liveness 属性则需在时序逻辑（LTL/CTL）中引入 eventually 算子，并依赖系统模型中的公平性假设（如弱公平性 WF）和超时边界。

4. **工程设计的映射**：Safety 通常通过冗余、仲裁（quorum）、 fencing token、WAL、复制日志等机制保证；Liveness 则依赖超时（timeout）、指数退避重试（exponential backoff）、leader election、故障检测器（failure detector）和 gossip 协议来推动系统收敛。

5. **核心纪律**：在规格说明阶段，必须将每一条正确性需求显式标注为 Safety 或 Liveness；在故障模型变化（同步/异步、崩溃停止/拜占庭故障）时，分别检查两类属性是否仍然保持，并避免用牺牲 Liveness 的代价去"修补" Safety 漏洞，反之亦然。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: Amazon Dynamo 的最终一致性与向量时钟
- **问题**: 分布式键值存储在网络分区下如何同时保证高可用与可接受的正确性。
- **方法论的使用**: Kleppmann 用 Safety/Liveness 框架分析 Dynamo 的设计选择：
  - **Safety**: Dynamo 牺牲强一致性（线性一致性是一种 Safety 属性），允许分区期间读取返回可能过时的数据；但保留"不会返回无法解析的冲突数据"这一 Safety 属性——通过向量时钟检测并发写入冲突。
  - **Liveness**: 优先保证可用性（availability），即任何节点在分区期间都能响应读写请求；冲突解决被推迟到读取阶段，由应用层合并，确保系统"最终"收敛到一致状态（eventual consistency 是一种 Liveness 属性）。
- **结论**: 在 CAP 权衡中，Dynamo 显式选择分区时牺牲 Safety（强一致性）以保全 Liveness（可用性），同时保留最低限度的 Safety（冲突可检测）。
- **结果**: 该设计支撑了 Amazon 购物车服务的高可用，成为 AP 系统的经典范式。

### 案例 2: Google MapReduce 的容错机制
- **问题**: 大规模批处理作业在节点频繁故障时如何保证计算正确性且最终完成。
- **方法论的使用**:
  - **Safety**: MapReduce 通过不可变输入数据、确定性 Map/Reduce 函数和原子性任务提交保证"不会输出错误结果"——失败任务的重执行不会产生不一致的输出（幂等性即 Safety）。
  - **Liveness**: 通过主节点（JobTracker）的心跳检测和超时机制，识别慢节点/故障节点并重新调度任务；通过 speculative execution 保证作业"最终"完成。
- **结论**: MapReduce 将 Safety 寄托于不可变数据和确定性计算，将 Liveness 寄托于超时-重试-重调度机制，两者解耦设计。
- **结果**: 非分布式专家也能写出可扩展的批处理程序，框架自动处理容错细节。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **分布式一致性选型困境**
   用户正在设计分布式数据库/缓存/消息系统，纠结"该选 CP 还是 AP"、"要不要上 Paxos/Raft"、"最终一致性够不够用"。此时需要用 Safety/Liveness 框架拆解：哪些需求是"不能出错"（Safety），哪些需求是"最终要对"（Liveness），并据此选择一致性级别和共识协议。

2. **并发协议的正确性验证**
   用户在实现分布式锁、两阶段提交、Saga、TCC 或自定义共识变体，需要向团队或自己证明"这个协议不会脑裂"（Safety）以及"这个协议不会永久死锁"（Liveness）。此时需要引入形式化定义、不变量推导和超时机制分析。

3. **超时与重试机制设计**
   用户在配置 RPC 超时、熔断策略、指数退避、心跳间隔，不确定"超时设多长"、"重试会不会放大故障"。这本质上是 Liveness 属性的工程实现：如何在异步网络中保证"好事最终发生"，同时不破坏 Safety（如重复请求导致双花）。

4. **故障注入与混沌测试规划**
   用户计划做 Chaos Engineering，需要明确测试目标。Safety 测试关注"注入故障后系统是否违反了不变量"；Liveness 测试关注"注入故障后系统是否能在有限时间内恢复"。

### 语言信号 (用户的话里出现这些就应激活)

- "系统会不会脑裂 / 双花 / 丢数据？"
- "这个设计最终能不能保证一致性 / 可用性 / 共识达成？"
- "我该选强一致性还是最终一致性？"
- "超时和重试怎么设计才不会出问题？"
- "怎么向团队证明这个分布式锁是正确的？"
- "CAP 定理在我们的场景里怎么落地？"
- "模型检查 / TLA+ 能验证什么，不能验证什么？"

### 与相邻 skill 的区分

- 与 `cap-pacelc-decision-matrix` 的区别: CAP/PACELC 聚焦一致性-可用性-延迟的权衡决策矩阵，而本 skill 深入正确性的形式化分类（Safety vs Liveness），解释"为什么 CAP 中一致性常对应 Safety、可用性对应 Liveness"，并提供验证方法（模型检查、不变量、超时机制）。
- 与 `concurrency-control-pessimistic-optimistic` 的区别: 并发控制框架聚焦锁策略与隔离级别的工程选型，而本 skill 关注并发/分布式协议的正确性属性分类与形式化证明思路。
- 与 `fault-tolerance-from-unreliable-parts` 的区别: 容错原则强调用冗余机制构建可靠系统，而本 skill 提供分析这些机制是否正确的分类语言（Safety/Liveness）和验证方法。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后，agent 应按以下步骤执行:

1. **识别用户场景中的正确性需求并分类为 Safety 或 Liveness**
   - 完成标准: 列出用户提到的所有正确性需求，并逐条标注为 Safety（nothing bad happens）或 Liveness（something good eventually happens）。
   - 判停条件: 若用户场景不涉及任何正确性/一致性/可用性/容错需求，则跳到步骤 5（边界说明），建议不使用本 skill。

2. **将需求映射到形式化定义与具体实例**
   - 完成标准: 对每条 Safety，给出其形式化表述（如不变量、前置/后置条件、历史序列约束）和书内/业界经典实例（如 uniqueness、linearizability、read-your-writes）；对每条 Liveness，给出其形式化表述（如 eventually、leads-to、fairness）和实例（如 eventual consistency、consensus termination、availability）。

3. **推荐验证方法或工程机制**
   - 完成标准:
     - Safety: 推荐不变量推导、归纳证明、TLA+/SPIN 模型检查、fencing token、quorum、WAL、复制日志等机制。
     - Liveness: 推荐超时设计（timeout）、指数退避重试、心跳/故障检测器、leader election、进度证明（progress argument）或 LTL 验证。
   - 判停条件: 若用户已明确指定某种验证工具或工程框架，则优先围绕该工具展开，不强行推荐其他方案。

4. **指出 Safety 与 Liveness 之间的耦合与冲突，并给出设计建议**
   - 完成标准: 明确说明用户场景中是否存在"为了保 Safety 而牺牲 Liveness"或"为了保 Liveness 而放松 Safety"的权衡（如 CAP 中的 CP vs AP），并给出 Kleppmann 式的务实建议：优先保证核心 Safety，对非核心 Safety 可通过降级策略换取 Liveness。

5. **引用反例与边界条件，提示常见陷阱**
   - 完成标准: 引用 counter-examples（ce08 忽略网络分区、ce09 过早优化一致性牺牲可用性），警告用户：
     - 不要假设网络可靠（忽略 Safety 漏洞的来源）；
     - 不要对所有操作强制线性一致性（误用 Safety 导致 Liveness 崩溃）。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 纯编程语法或 API 使用问题（如"Python 的 threading.Lock 怎么用"）。
- 单一节点的本地算法优化，不涉及并发或分布式正确性（如"如何加速单线程排序"）。
- 不涉及正确性分类的通用性能调优（如"如何降低 P99 延迟"，除非同时涉及超时与重试的 Liveness 设计）。
- 纯数学归纳法教学，不绑定到系统属性（如"如何证明斐波那契数列通项"）。

### 作者在书中警告的失败模式

- **ce08: 忽略网络分区（假设网络可靠）**
  设计者假设节点间通信总是成功，未处理超时、丢包和分区，导致在真实网络故障时数据不一致、服务雪崩或脑裂。Safety 属性（如唯一性、一致性）在分区场景下会瞬间被击穿，若未预先定义分区行为策略，系统将以不可预测的方式失效。

- **ce09: 过早优化一致性而牺牲可用性**
  在不需要强一致性的业务场景（如社交点赞、评论数、商品库存预估）强制使用线性一致性或分布式事务，导致系统延迟高、可用性差、扩展困难。这是对 Safety/Liveness 权衡的误用：将高成本的 Safety 保证强加于不需要它的应用，直接损害了 Liveness（可用性）。

### 作者的盲点 / 时代局限

- Kleppmann 对 Safety/Liveness 的讨论以异步网络和崩溃停止模型为主，对拜占庭故障（Byzantine faults）和 LLM 时代的新型 AI 基础设施（如向量数据库的一致性、模型推理服务的流式输出正确性）覆盖不足。
- 对"人"的因素（如运维人员在分区时的手动切换决策、组织政治对 CAP 选择的影响）讨论极少，假设理想化的工程师环境。
- 形式化验证工具（TLA+、Coq、Ivy）在工业界的落地门槛被低估，实际团队中掌握这些工具的人员稀缺。

### 容易混淆的邻近方法论

- **CAP 定理 vs Safety/Liveness**: CAP 是分布式系统的一个特定不可能结果（在分区时无法同时满足一致性和可用性），而 Safety/Liveness 是更普适的正确性分类语言。CAP 中的 C 更接近线性一致性（一种 Safety），A 更接近可用性（一种 Liveness），但 Safety/Liveness 的适用范围远超 CAP（如并发协议、操作系统、硬件验证）。
- **ACID 中的 Consistency vs Safety**: ACID 的 Consistency 通常指应用级不变量（一种 Safety），但分布式一致性模型（Linearizability、Sequential Consistency、Eventual Consistency）中，有些属于 Safety，有些属于 Liveness。讨论时需明确语境。

---

## 相关 skills

- depends-on: formal-proof-invariant-verification
- contrasts-with: (none)
- composes-with: concurrency-strategy-selection, distributed-consistency-decision


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 待测 (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
