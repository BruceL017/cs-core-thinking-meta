---
name: data-modeling-decision
description: |
  当用户面临数据库/存储选型、schema 设计、查询语言选择或多模型共存架构决策时激活。
  典型信号：讨论关系型 vs 文档型 vs 图数据库选型；提到 " impedance mismatch "、"schema 灵活"、"多对多关系"、"图遍历"、"Cypher/SPARQL"；需要为不同查询模式选择存储引擎或列存方案。
  不适用于：纯 SQL 语法调试、单一数据库的运维问题、与数据模型无关的分布式一致性讨论。
source_book: 《Designing Data-Intensive Applications》Martin Kleppmann; 《Readings in Database Systems》Stonebraker et al.
source_chapter: "DDIA Ch.2 / RedBook Ch.1 & Ch.9"
tags: [data-modeling, query-languages, relational-model, document-model, graph-model, polyglot-persistence, schema-design]
related_skills: []
---

# 数据模型抽象与查询语言选择的决策框架

## R — 原文 (Reading)

> "The best choice depends on the relationships between data items... Relational models are good for many-to-one and many-to-many relationships; document models are good for self-contained data with a tree structure; graph models are good for highly interconnected data." (DDIA Ch.2)
> "Nobody ever seems to learn anything from history... A decade ago, the buzz was all XML. Vendors were intent on adding XML to their relational engines." (RedBook Ch.1)
>
> — Kleppmann, DDIA Ch.2; Stonebraker et al., RedBook Ch.1

---

## I — 方法论骨架 (Interpretation)

这个框架解决的核心问题是：给定一组数据和查询需求，如何选择最合适的数据模型与查询语言组合。

决策依赖于三个维度的分析：
1. **数据关系模式**：实体之间是一对多、多对一还是多对多？数据是否自包含、具有清晰的层次结构？
2. **查询模式**：是以点查询和事务为主，还是需要复杂的连接、遍历或分析聚合？声明式语言（如 SQL）是否足以表达，还是需要专门的图查询语言（如 Cypher）或逻辑语言（如 Datalog）？
3. **演化需求**：schema 是否需要快速变化？强 schema 的治理成本与弱 schema 的数据质量风险如何权衡？

关系模型擅长处理规范化的多对多关系和复杂分析查询；文档模型适合自包含、模式灵活的数据；图模型（属性图、RDF）在高度互联数据和路径遍历场景下具有不可替代性。现实中，单一模型往往无法覆盖所有访问模式，因此多模型共存（polyglot persistence）是常见策略。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: LinkedIn 个人资料查询的缓存与数据库分层设计 (c08)
- **问题**: LinkedIn 的个人资料页面请求涉及多种访问模式——点查询获取常用字段、全文搜索、以及事务型更新。
- **方法论的使用**: 作者没有试图用单一数据库解决所有问题，而是为每种访问模式选择了最合适的存储抽象：Voldemort 键值缓存处理点查询、ESPI 搜索索引处理全文检索、Oracle 数据库存储权威事务数据。
- **结论**: 复杂查询场景下，单一数据库往往无法满足所有访问模式。
- **结果**: 通过 Databus 变更日志保持层间最终一致，实现了 polyglot persistence 的务实落地。

### 案例 2: C-Store / Vertica 的列式存储与投影排序设计 (c13)
- **问题**: 传统行式存储在分析型查询（大量聚合、扫描少数列）中 I/O 效率极低。
- **方法论的使用**: C-Store 打破行式存储抽象，将同一列的数据连续存储，并允许同一张逻辑表维护多个按不同列排序的"投影"（projections），以优化不同查询负载。
- **结论**: 数据布局（物理存储）必须与访问模式（分析型 vs 事务型）相匹配。
- **结果**: 列式存储配合向量化执行，将分析查询的 I/O 和压缩效率提升数个数量级。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **数据库/存储选型会议**：团队要在 PostgreSQL、MongoDB、Neo4j 之间做选择，需要基于数据特征和查询模式进行系统化论证。
2. **Schema 演进与治理争议**：产品经理要求快速迭代字段，DBA 要求强 schema 约束，需要权衡文档模型的灵活性与关系模型的数据质量保障。
3. **多模型架构设计**：系统同时需要事务处理、图遍历和全文搜索，正在考虑是否引入多个专用存储，以及如何管理数据一致性和同步。

### 语言信号 (用户的话里出现这些就应激活)

- "我们该用关系型还是文档型数据库？"
- "这个数据有很多多对多关系，是不是该上图数据库？"
- "schema 变化太快，用 MongoDB 是不是更好？"
- "我们需要支持 Cypher 查询 / SPARQL / Datalog"
- "同一个数据既要做 OLTP 又要做图分析，能不能放在一个库里？"
- "列存和行存怎么选？要不要多个投影？"

### 与相邻 skill 的区分

- 与 `quantitative-performance-analysis` 的区别: 本 skill 聚焦于"数据如何被组织和查询"的抽象选择，而非具体的性能测量与瓶颈分析。虽然会涉及负载特征，但不进行 profiling 或基准测试。
- 与 `distributed-consistency-cap-pacelc` 的区别: 本 skill 处理的是单机/集群层面的数据模型与查询语言选择，而非分布式副本之间的一致性级别与容错策略。
- 与 `interface-separation-evolution` 的区别: 本 skill 关注的是数据模型本身的抽象边界，而非通用软件接口的版本治理与封装策略。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后，agent 应按以下步骤执行:

1. **绘制数据关系草图**
   - 与用户一起识别核心实体及其关系类型（一对一、一对多、多对一、多对多）。
   - 完成标准: 能清晰说出"哪些关系是层次化的、自包含的"，"哪些关系是高度互联的、需要遍历的"。

2. **分析查询模式与访问路径**
   - 列出主要的读查询和写查询：是点查询、范围扫描、连接聚合，还是图遍历、全文搜索？
   - 评估声明式查询的充分性：SQL 能否自然表达？是否需要递归查询（如 SQL 的 WITH RECURSIVE）或专门的图查询语言（Cypher、Gremlin、SPARQL、Datalog）？
   - 判停条件: 若用户仅关心单一数据库的 SQL 语法问题，则跳到步骤 B（边界说明），不继续执行本 skill。

3. **评估 schema 灵活性与治理需求**
   - 判断数据模式是否稳定：是否需要严格的类型约束、外键关系和变更审计？
   - 完成标准: 明确选择强 schema（关系型/数据仓库）或弱 schema（文档型/键值型）的核心理由。

4. **推荐数据模型组合与查询语言**
   - 基于前三步，给出具体推荐：
     - 多对一/多对多 + 复杂分析 → 关系模型 + SQL
     - 自包含文档 + 快速迭代 → 文档模型 + 文档查询语言
     - 高度互联 + 路径遍历 → 属性图模型 + Cypher/Gremlin，或 RDF + SPARQL/Datalog
     - 混合负载 → 多模型共存策略（polyglot persistence），并说明层间同步机制
   - 完成标准: 用户能明确说出"我们选择 X 模型，因为 Y 查询模式与 Z 数据关系高度匹配"。

5. **引用反例进行风险警示**
   - 明确警告以下失败模式：
     - 不要为追逐技术潮流而将高度关系化数据硬塞入文档/图模型（RedBook 的 XML 教训）。
     - 不要假设单一数据库能最优地服务所有访问模式（LinkedIn 案例的启示）。
     - 不要忽视 schema 治理：弱 schema 在快速上线后可能积累严重的数据质量问题。
   - 完成标准: 用户能复述至少一个需要避免的具体陷阱。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 用户只是询问某个具体 SQL/NoSQL 语句的写法或调试报错。
- 讨论与数据模型无关的纯分布式一致性、事务隔离级别或共识算法问题。
- 用户已经明确选型且方案无可争议，只是需要部署配置建议。

### 作者在书中警告的失败模式

- **重复历史的陷阱** (RedBook Ch.1): 十年前业界狂热追逐 XML，厂商强行将 XML 加入关系引擎；今天类似的风险是用 JSON 文档存储本质上高度关系化的数据。数据模型选择应基于数据结构的内在特征，而非技术潮流。
- **单一数据源的盲目信任** (ce11 / DDIA Ch.10): "Garbage in, garbage out." 在多模型共存架构中，若缺乏显式的 schema 验证、数据契约和监控，错误数据会在 pipeline 中静默传播，直到在业务端造成可见损失。

### 作者的盲点 / 时代局限

- 这些教材写于 2017–2021 年，对于向量数据库（vector DB）、AI 基础设施中的 embedding 存储、以及无服务器架构下的数据模型讨论不足。
- 对 LLM 时代新兴的"图+RAG"混合架构（如知识图谱增强检索）没有直接覆盖。

### 容易混淆的邻近方法论

- **与查询优化器框架 (f15) 的区别**: 查询优化器讨论的是"在已选定关系模型后，如何生成最优执行计划"；本 skill 讨论的是"是否应该选择关系模型"。
- **与抽象屏障框架 (f16) 的区别**: 抽象屏障框架强调接口与实现的分离；本 skill 强调的是数据本身的结构特征与查询语言之间的匹配。

---

## 相关 skills

- depends-on: abstraction-barrier-evolution
- contrasts-with: (none)
- composes-with: query-optimization-cbo, data-system-reliability-assessment


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 待测 (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
