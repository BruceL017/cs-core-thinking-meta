---
name: cs-core-thinking-router
description: |
  当用户讨论任何与计算机科学核心思维相关的系统设计、性能优化、并发控制、网络协议、数据存储、持久化、形式化验证、抽象设计、调度策略、查询优化或数据结构选型时，**必须立即调用本 skill**，不要试图直接跳到底层 skills。
  本 skill 是 18 个 CS core thinking skills 的唯一入口，负责在用户意图与具体技能链之间做路由，防止遗漏前置分析（如跳过 quantitative-performance-analysis）或错误匹配领域。
  典型触发场景（包括但不限于）：系统架构设计评审、性能瓶颈根因分析、缓存/存储策略选型、数据库事务隔离级别选择、分布式锁与共识协议设计、TCP/丢包/拥塞/滑动窗口问题、概率数据结构（Bloom filter/Treap）选型、算法正确性证明与不变量推导、操作系统虚拟化与 CPU/内存调度、抽象边界与接口演化、DSL/解释器设计。
  边界模糊场景也必须触发：如"分布式数据库查询慢"可能同时涉及查询优化和网络分区，此时 router 会判断主诉并给出调用链；"Redis 缓存穿透"可能涉及概率数据结构或可靠性评估，同样先走 router。
  不适用于：纯前端 UI 设计、日常运维操作（如重启服务/查日志）、无上下文的单一算法题求解。
---

# CS Core Thinking Router

## 使命

在 18 个 CS core thinking skills 中，**用一次快速分类**确定用户问题的根领域，输出一条明确的 skill 调用链，避免遗漏前置分析或错误跳过关键步骤。

## 决策路由树

按以下顺序匹配用户问题，命中即停止，返回对应的调用链。

### 1. 性能、成本、加速、量化分析
**关键词**：CPU 时间、Amdahl 定律、缓存未命中、加速比、吞吐量、延迟、p99、过早优化、负载建模

**调用链**：
1. `quantitative-performance-analysis`（先做量化拆解：CPU Time = IC × CPI × Cycle Time）
2. 根据子问题分支：
   - 缓存/数据布局/存储局部性 → `locality-driven-optimization`
   - 查询优化/执行计划选择 → `query-optimization-cbo`
   - 调度策略/OS 调度器设计 → `scheduling-policy-design`
   - 海量数据/内存受限/允许近似 → `probabilistic-data-structure-tradeoff`

### 2. 网络、传输、协议设计
**关键词**：TCP、丢包、ACK、重传、超时、滑动窗口、拥塞控制、cwnd、分层、端到端原则、网关、路由器

**调用链**：
1. `layered-end-to-end-decision`（先决定功能该放哪一层）
2. 若涉及可靠传输/丢包恢复 → `reliable-transport-arq`
3. 若涉及带宽分配/公平性/网络拥塞 → `congestion-control-aimd`

### 3. 并发控制、事务隔离、锁策略
**关键词**：悲观锁、乐观锁、MVCC、SSI、两阶段锁、死锁、事务隔离级别、竞态条件、读写比例

**调用链**：
1. `concurrency-strategy-selection`（评估悲观 vs 乐观策略）
2. 若需形式化验证正确性 → `safety-liveness-properties`

### 4. 分布式系统、一致性、共识
**关键词**：CAP、PACELC、线性一致性、最终一致性、Raft、Paxos、2PC、Saga、脑裂、复制滞后、分布式锁

**调用链**：
1. `distributed-consistency-decision`（CAP/PACELC 双模决策）
2. 若涉及协议正确性分类 → `safety-liveness-properties`

### 5. 数据系统、存储、可靠性、持久化
**关键词**：数据库选型、WAL、日志、ARIES、LSM-Tree、COW、RAID、崩溃恢复、RSM、SLA/SLO、容灾、备份

**调用链**：
1. `data-system-reliability-assessment`（R/S/M 三维度评估）
2. 根据子问题分支：
   - 崩溃恢复/持久化协议 → `crash-recovery-persistence-design`
   - 数据模型选型（关系/文档/图） → `data-modeling-decision`

### 6. 形式化验证、正确性证明、安全属性
**关键词**：数学归纳法、不变量、状态机、并发安全、死锁证明、Safety、Liveness、边界条件验证

**调用链**：
1. `formal-proof-invariant-verification`（归纳法与不变量原理）
2. 若涉及并发/分布式协议属性分类 → `safety-liveness-properties`

### 7. 抽象设计、虚拟化、语言与元语言
**关键词**：接口与实现分离、抽象边界、虚拟内存、CPU 虚拟化、解释器、DSL、同像性、元循环求值

**调用链**：
1. `abstraction-barrier-evolution`（抽象边界演化）
2. 根据子问题分支：
   - 虚拟化/资源管理 → `virtualization-resource-abstraction`
   - 数据模型/数据库 schema 设计 → `data-modeling-decision`
   - 解释器/DSL/元语言 → `metalinguistic-abstraction`

### 8. 概率数据结构、空间-时间-正确性权衡
**关键词**：Bloom filter、Treap、R-tree、假阳性、Count-Min Sketch、哈希碰撞、生日悖论、近似最近邻

**调用链**：
1. `probabilistic-data-structure-tradeoff`
2. 若需严格证明误差界 → `formal-proof-invariant-verification`

---

## 执行步骤

1. **提取用户问题的核心语义**
   - 忽略代码细节请求（如"帮我写这段代码"），聚焦背后的设计/决策问题。
   - 识别用户真正在问的 belongs to 上述 8 个根领域中的哪一个。

2. **匹配路由树并生成调用建议**
   - 使用下方输出模板，给出 PRIMARY（首要调用）、SECONDARY（补充调用）、DEPENDS_ON（必须先调用的前置 skill）。
   - 若用户问题同时跨多个领域（如"分布式数据库的性能优化"），优先按**问题的主语**路由：如果是"分布式一致性怎么选"，走领域 4；如果是"这个分布式数据库延迟高怎么优化"，走领域 1 → 再补 5/4。

3. **边界检查**
   - 若问题属于纯前端 UI、日常运维操作、或无上下文的算法题，明确输出 `"不适用 CS core thinking skills"`。
   - 若用户问题模糊，先追问 1–2 个澄清问题，再进行路由。

---

## 输出模板

```markdown
## 路由结果

**主领域**: <8 个根领域之一>

**调用顺序**:
1. <skill-name-1> — 原因：<一句话>
2. <skill-name-2> — 原因：<一句话>
3. <skill-name-3> — 原因：<一句话>（可选）

**避免调用**: <若有与用户问题相关但不合适的 skill，说明原因>

**若信息不足，需要澄清**: <问题列表，或写"无">
```

---

## 快速对照表

| 用户说的话 | 路由结果 |
|---|---|
| "这个架构能不能撑住流量翻倍？" | 1. quantitative-performance-analysis → 2. data-system-reliability-assessment |
| "我们该用悲观锁还是乐观锁？" | 1. concurrency-strategy-selection |
| "RAID-5 小写惩罚怎么量化？" | 1. crash-recovery-persistence-design（前置: quantitative-performance-analysis） |
| "Bloom filter 假阳性率怎么算？" | 1. probabilistic-data-structure-tradeoff |
| "TCP 丢包怎么恢复？" | 1. layered-end-to-end-decision → 2. reliable-transport-arq |
| "这个功能放客户端还是服务端？" | 1. layered-end-to-end-decision |
| "如何证明这个并发算法正确？" | 1. formal-proof-invariant-verification → 2. safety-liveness-properties |
| "关系型数据库和 MongoDB 怎么选？" | 1. data-modeling-decision（前置: abstraction-barrier-evolution） |

---

## 18 个技能全名索引

- `quantitative-performance-analysis` — 定量性能分析框架
- `locality-driven-optimization` — 局部性驱动的优化决策
- `probabilistic-data-structure-tradeoff` — 概率数据结构的空间-时间-正确性权衡
- `scheduling-policy-design` — 调度策略设计框架
- `query-optimization-cbo` — 基于代价的查询优化
- `layered-end-to-end-decision` — 分层抽象与端到端原则决策
- `reliable-transport-arq` — 不可靠信道上的可靠传输
- `congestion-control-aimd` — 拥塞控制 AIMD 框架
- `concurrency-strategy-selection` — 并发控制策略选择
- `distributed-consistency-decision` — 分布式一致性决策
- `formal-proof-invariant-verification` — 归纳证明与不变量验证
- `safety-liveness-properties` — Safety 与 Liveness 正确性分类
- `data-system-reliability-assessment` — 数据系统 RSM 可靠性评估
- `crash-recovery-persistence-design` — 崩溃恢复与持久化设计
- `data-modeling-decision` — 数据建模决策框架
- `abstraction-barrier-evolution` — 抽象边界演化策略
- `virtualization-resource-abstraction` — 虚拟化资源抽象
- `metalinguistic-abstraction` — 元语言抽象框架
