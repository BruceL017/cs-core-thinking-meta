# CS Core Thinking Meta-Framework

> 从 8 本计算机科学经典教材中蒸馏出的 18 个可执行 Agent Skills + 1 个决策路由树。

## 使用方式

这 18 个 skills 采用**决策路由树**组织。Agent 在接收到相关问题时，**应先调用** `cs-core-thinking-router`，由 router 根据用户语义输出一条明确的 skill 调用链，再按顺序激活具体 skills。

这能避免两类常见错误：
- **遗漏前置分析**：如跳过 `quantitative-performance-analysis` 直接调 `concurrency-strategy-selection`
- **错误匹配**：如把网络拥塞问题误当作缓存局部性问题处理

## 18 个核心 Skills

| Skill | 核心问题 | 来源书籍 |
|---|---|---|
| **quantitative-performance-analysis** | 如何用量化方法避免感性优化和过早优化？ | CAQA, DDIA |
| **locality-driven-optimization** | 如何利用时间/空间局部性指导缓存、存储和算法设计？ | CAQA, OSTEP, AADS |
| **probabilistic-data-structure-tradeoff** | 如何在资源受限时权衡空间-时间-正确性，使用概率数据结构？ | MCS, AADS |
| **scheduling-policy-design** | 如何根据 workload 特征推导调度策略（CFS/MLFQ/lottery）？ | OSTEP |
| **query-optimization-cbo** | 数据库查询优化器如何基于代价选择执行计划（CBO/System R）？ | RedBook |
| **layered-end-to-end-decision** | 功能应放在系统哪一层实现（分层 vs 端到端原则）？ | CN, SICP, OSTEP |
| **reliable-transport-arq** | 不可靠信道上如何设计可靠传输（ACK/重传/超时/序列号）？ | CN, OSTEP |
| **congestion-control-aimd** | 共享网络中如何公平高效地分配带宽（AIMD/cwnd）？ | CN, OSTEP |
| **concurrency-strategy-selection** | 悲观锁 vs 乐观锁/MVCC/SSI 如何根据 workload 选择？ | OSTEP, RedBook, DDIA |
| **distributed-consistency-decision** | 分布式系统中的一致性级别如何选择（CAP/PACELC）？ | DDIA, RedBook |
| **formal-proof-invariant-verification** | 如何用归纳法和不变量原理验证系统正确性？ | MCS, SICP, OSTEP |
| **safety-liveness-properties** | 如何用 Safety/Liveness 分类并发与分布式协议的正确性属性？ | MCS, OSTEP, DDIA |
| **data-system-reliability-assessment** | 如何系统化评估后端系统的 R/S/M 三维度？ | DDIA |
| **crash-recovery-persistence-design** | 文件系统/数据库如何选择崩溃恢复策略（WAL/LFS/COW）？ | OSTEP, RedBook, DDIA |
| **data-modeling-decision** | 关系型/文档型/图数据库如何选型与建模？ | DDIA, RedBook |
| **abstraction-barrier-evolution** | 如何设计可长期演化的抽象边界和接口？ | SICP, RedBook, DDIA |
| **virtualization-resource-abstraction** | 如何通过虚拟化抽象管理 CPU/内存/磁盘资源？ | OSTEP, SICP |
| **metalinguistic-abstraction** | 如何通过元语言抽象（解释器/DSL/同像性）控制复杂度？ | SICP |

## 来源书籍

- **CAQA**: *Computer Architecture: A Quantitative Approach* — Hennessy & Patterson
- **CN**: *Computer Networking: A Top-Down Approach* — Kurose & Ross
- **DDIA**: *Designing Data-Intensive Applications* — Martin Kleppmann
- **OSTEP**: *Operating Systems: Three Easy Pieces* — Arpaci-Dusseau
- **RedBook**: *Readings in Database Systems* — Stonebraker et al.
- **SICP**: *Structure and Interpretation of Computer Programs* — Abelson & Sussman
- **MCS**: *Mathematics for Computer Science* — Lehman et al.
- **AADS**: *Advanced Algorithms and Data Structures* — La Rocca

## 路由决策树（摘要）

```
cs-core-thinking-router
├── 性能/量化分析 → quantitative-performance-analysis
│   ├── 缓存/局部性 → locality-driven-optimization
│   ├── 查询优化 → query-optimization-cbo
│   ├── 调度策略 → scheduling-policy-design
│   └── 概率结构 → probabilistic-data-structure-tradeoff
├── 网络/协议 → layered-end-to-end-decision
│   ├── 可靠传输 → reliable-transport-arq
│   └── 拥塞控制 → congestion-control-aimd
├── 并发控制 → concurrency-strategy-selection
│   └── 形式化验证 → safety-liveness-properties
├── 分布式一致性 → distributed-consistency-decision
│   └── 形式化验证 → safety-liveness-properties
├── 数据/存储/可靠性 → data-system-reliability-assessment
│   ├── 崩溃恢复 → crash-recovery-persistence-design
│   └── 数据建模 → data-modeling-decision
├── 形式化验证 → formal-proof-invariant-verification
│   └── 安全/活性属性 → safety-liveness-properties
├── 抽象/虚拟化/语言 → abstraction-barrier-evolution
│   ├── 虚拟化 → virtualization-resource-abstraction
│   ├── 数据建模 → data-modeling-decision
│   └── 元语言 → metalinguistic-abstraction
└── 概率数据结构 → probabilistic-data-structure-tradeoff
```

## 项目结构

```
books/cs-core-thinking-meta/
├── BOOK_OVERVIEW.md          # 阶段 0: 跨书元框架理解
├── INDEX.md                  # 完整索引 + Mermaid 依赖图
├── README.md                 # 本文件
├── candidates/               # 阶段 1 提取的候选池
│   ├── frameworks.md         # 23 个框架
│   ├── principles.md         # 24 条原则
│   ├── cases.md              # 22 个案例
│   ├── counter-examples.md   # 30 个失败模式
│   └── glossary.md           # 45 个术语
├── rejected/                 # 阶段 1.5 淘汰的候选
└── <skill-name>/             # 18 个原子 skill
    ├── SKILL.md
    └── test-prompts.json     # Darwin 兼容测试集
```

## 审计统计

- **Books**: 8
- **Candidate units extracted**: 144
- **Skills distilled**: 18
- **Meta-router**: 1 (`cs-core-thinking-router`)
- **Triple-verified**: V1 ✓ / V2 ✓ / V3 ✓
- **Darwin test prompts**: 18 × (3 should_trigger + 2 should_not_trigger + 1 edge_case)

## License

MIT
