---
name: query-optimization-cbo
description: |
  当用户讨论数据库查询性能调优、执行计划选择、连接顺序优化、基数估计偏差、统计信息更新、CBO/ RBO 选型、SQL 引擎架构设计，或出现"EXPLAIN 计划不对""ANALYZE 后变快""优化器选了嵌套循环""连接顺序导致查询爆炸"等信号时调用。
  不适用于：纯 SQL 语法问题、ORM 使用教程、特定数据库的安装配置、无执行计划信息的泛泛性能抱怨。
source_book: 《Readings in Database Systems》Stonebraker et al.
source_chapter: "RedBook Ch.3 & Ch.7 — Query Optimization"
tags: [query-optimization, cost-based-optimizer, database-systems, execution-plan, cardinality-estimation]
related_skills: []
---

# 查询优化器的成本模型与搜索空间框架

## R — 原文 (Reading)

> "Query optimization is one of the signature components of database technology—the bridge that connects declarative languages to efficient execution." / "No query optimizer is truly producing 'optimal' plans. First, they all use estimation techniques to guess at real plan costs, and it's well known that errors in these estimation techniques can balloon. Second, optimizers use heuristics to limit the search space of plans they choose, since the problem is NP-hard."
>
> — Stonebraker et al., RedBook Ch.3 & Ch.7 — Query Optimization

---

## I — 方法论骨架 (Interpretation)

数据库查询优化器的核心任务，是把声明式的 SQL 转化为高效的物理执行计划。它通过三个相互依赖的子系统完成这一任务：

1. **统计信息与代价估算（Cost Estimation）**：基于表基数、列直方图、索引选择性等统计信息，估算每个操作符（扫描、连接、排序）的 I/O 与 CPU 成本。
2. **关系等价与搜索空间（Plan Space）**：利用关系代数等价规则（选择下推、投影消除、连接重排序）生成逻辑上等价但执行效率迥异的候选计划。
3. **基于代价的搜索算法（Cost-Based Search）**：在庞大的计划空间中用动态规划（System R 自底向上）或分支定界（Volcano/Cascades 自顶向下）寻找最低代价计划，同时用启发式剪枝控制优化时间。

CBO 的本质不是"找到最优计划"，而是在"统计误差、搜索时间、计划质量"三者之间做可控的权衡。优化器依赖的统计信息会过时，基数估计误差会在多表连接时指数级放大（误差传播），而连接顺序搜索空间随表数呈阶乘增长，因此所有商用优化器都依赖启发式边界（如禁止笛卡尔积、限制 Bushy Tree 深度）来使问题可解。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: System R 查询优化器的基于代价的决策
- **问题**: 早期的关系数据库依赖手工选择访问路径，SQL 的声明性优势被物理执行细节抵消。
- **方法论的使用**: Selinger 等人在 IBM System R 中首次将查询优化形式化为搜索问题：枚举等价查询计划，用统计信息（基数、索引选择性）估算 I/O 和 CPU 成本，选择代价最小的计划。
- **结论**: 基于代价的优化（CBO）将查询调优从手工艺术转变为可自动化的工程问题。
- **结果**: System R 的 CBO 框架成为现代关系型数据库（Oracle、PostgreSQL、SQL Server、Db2）查询优化器的共同祖先。

### 案例 2: C-Store / Vertica 的列式存储与投影排序设计
- **问题**: 分析型工作负载需要扫描大量列数据，传统行式存储的 I/O 效率极低。
- **方法论的使用**: C-Store 打破单一物理视图的假设，允许同一张逻辑表存在多个按不同列排序的"投影"（projections）。优化器根据查询模式选择访问哪个投影，将列式压缩、向量化执行与投影选择整合进 CBO 框架。
- **结论**: 物理存储布局应与访问模式匹配；为不同查询负载维护多个物理视图是 CBO 框架下的务实权衡。
- **结果**: 分析查询性能提升数个数量级，但增加了存储冗余和优化器在投影选择上的决策复杂度。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **执行计划异常**：用户贴出 EXPLAIN / EXPLAIN ANALYZE 输出，发现优化器选择了明显低效的计划（如大表嵌套循环连接、未使用可用索引、错误的连接顺序），需要诊断是统计信息缺失、代价模型偏差还是搜索空间限制导致的。
2. **统计信息维护策略**：用户在讨论数据库运维时，需要决定 ANALYZE / UPDATE STATISTICS 的频率、采样率、自动收集触发条件，以及统计信息过期对 CBO 决策的影响。
3. **SQL 引擎/数仓架构设计**：用户在构建或选型 SQL 引擎（Spark SQL、Presto、ClickHouse、DuckDB、TiDB）时，需要理解其优化器架构（RBO vs CBO、Volcano/Cascades、自适应优化）以及连接算法（Nested Loop / Hash Join / Merge Join）的选择逻辑。

### 语言信号 (用户的话里出现这些就应激活)

- "EXPLAIN 显示优化器走了全表扫描，但明明有索引"
- "ANALYZE 之后查询突然变快了"
- "这个多表连接的顺序不对，怎么强制改？"
- "基数估计偏差太大，导致选了 hash join 而不是 merge join"
- "我们在自研 SQL 引擎，优化器该用 Cascades 还是动态规划？"
- "System R 的 Selinger 论文里连接顺序是怎么枚举的？"

### 与相邻 skill 的区分

- 与 `locality-driven-optimization` 的区别: 后者聚焦缓存、数据布局与存储层次；本 skill 聚焦查询优化器的计划搜索与代价估算，虽然 C-Store 投影选择涉及布局，但核心决策机制是 CBO 而非局部性原理。
- 与 `quantitative-performance-analysis` 的区别: 后者是通用的性能测量与瓶颈分析框架；本 skill 专门讨论数据库查询优化器内部的代价模型、基数估计和计划空间搜索算法。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

1. **诊断问题类型：统计信息、代价模型还是搜索空间？**
   - 完成标准: 明确用户遇到的是计划选择错误（CBO 选了坏计划）、计划生成失败（优化时间过长），还是引擎架构设计问题。
   - 判停条件: 若用户只是询问某条 SQL 的语法写法，则跳到步骤 B（边界），不深入 CBO 细节。

2. **分析执行计划与统计信息状态**
   - 完成标准:
     - 若用户提供了 EXPLAIN 输出，解析其中的操作符类型（Seq Scan / Index Scan / Hash Join / Nested Loop / Merge Join）、估计行数（rows）与实际行数的偏差。
     - 询问或推断统计信息的新鲜度（最后一次 ANALYZE 时间、采样率、是否有直方图）。
     - 指出基数估计误差可能的传播路径（如多表连接时独立假设失效导致估计值指数级偏离）。

3. **给出针对性优化或设计建议**
   - 完成标准:
     - 若统计信息过时：建议更新统计信息、提高采样率、收集多列统计信息（extended statistics）以修正关联列的独立假设。
     - 若代价模型偏差：解释当前数据库的代价参数（如 random_page_cost、seq_page_cost、cpu_tuple_cost）如何影响计划选择，建议微调或校准。
     - 若搜索空间受限：解释动态规划（DPsize / DPsub）的表数限制、启发式剪枝规则（如禁止笛卡尔积）、以及何时应考虑使用优化器提示（hint）或重写 SQL 来引导计划生成。
     - 若涉及引擎架构设计：对比 System R 自底向上 DP、Volcano 分支定界、Cascades 基于规则的搜索框架，建议根据团队资源和时间线选择实现路径。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 用户只是问 SQL 语法（如窗口函数用法、CTE 递归写法），没有涉及执行计划或优化器决策。
- 用户抱怨数据库慢，但没有任何 EXPLAIN 输出、表结构或统计信息，无法进行 CBO 层面的诊断。
- 讨论的是分布式事务协议（2PC/Paxos/Raft）或存储引擎（B-Tree/LSM-Tree）的实现细节，与查询优化器的计划选择无关。

### 作者在书中警告的失败模式

- **ce16: 在没有索引的情况下进行全表扫描**
  全表扫描的时间复杂度为 O(N)，随着数据量增长会耗尽 I/O 带宽并将热数据逐出缓存。CBO 在缺乏索引统计信息或代价参数设置不当时，可能错误地选择全表扫描而非索引扫描。
- **ce17: 忽略查询优化器的提示**
  优化器的成本模型基于统计信息。过时或缺失的统计信息会导致优化器选择嵌套循环连接处理大表、错误估计基数，生成执行时间高出数个数量级的计划。强制使用 hint 而不理解其适用条件也会在未来数据分布变化时造成计划退化。

### 作者的盲点 / 时代局限

- RedBook 中关于查询优化的经典论述主要基于传统关系型数据库（System R、Ingres、PostgreSQL），对现代云原生数仓（如 Snowflake 的自动集群、DuckDB 的激进压缩感知优化）和 AI 驱动的学习型查询优化（learned cardinality estimation、Neo、Balsa）覆盖不足。
- 对向量数据库、图查询优化（如 Neo4j 的基于模式匹配的优化器）以及流处理系统中的连续查询优化讨论较少。

### 容易混淆的邻近方法论

- 与索引设计/物理调优的区别: 索引设计是"给优化器提供更多选择"，而本 skill 关注的是"优化器如何在已有选择中做决策"。两者常配合，但属于不同层面。
- 与 SQL 重写的区别: SQL 重写是改变查询的语义等价形式以影响搜索空间，而本 skill 讨论的是优化器内部如何枚举和评估这些等价形式。

---

## 相关 skills

- depends-on: quantitative-performance-analysis, data-modeling-decision
- contrasts-with: (none)
- composes-with: locality-driven-optimization


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 待测 (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
