# 跨书常见失败模式速查表

> 从 8 本经典 CS 教材中提取的 30 个核心 counter-examples，按领域分类。各 skill 的 Boundary 段应引用此处编号（ce01–ce28）。

## 性能与体系结构

| 编号 | 失败模式 | 核心教训 | 绑定 Skill |
|---|---|---|---|
| ce01 | 缓存越大越好 | 命中时间增加可能抵消命中率收益 | locality-driven-optimization |
| ce02 | 流水线可以无限深 | 分支预测失败惩罚与功耗墙限制深度 | quantitative-performance-analysis |
| ce03 | Amdahl 定律不适用并行机 | 串行比例决定加速天花板 | quantitative-performance-analysis |
| ce04 | 单纯提高频率忽视功耗 | P ∝ V²f，热节流会反噬性能 | quantitative-performance-analysis |
| ce28 | 只追渐近最优不顾缓存局部性 | 链表 vs 数组扫描实际性能差 10–100 倍 | locality-driven-optimization, probabilistic-data-structure-tradeoff |

## 网络与传输

| 编号 | 失败模式 | 核心教训 | 绑定 Skill |
|---|---|---|---|
| ce05 | 隐藏终端导致 CSMA 失效 | 无线网络需要特殊冲突避免机制 | reliable-transport-arq |
| ce06 | 增加带宽就能解决所有延迟问题 | 传播延迟是物理下限，带宽不能消除 RTT | layered-end-to-end-decision |
| ce07 | 忽略网络层拥塞信号 | 端到端必须自己探测和响应拥塞 | congestion-control-aimd |

## 并发与一致性

| 编号 | 失败模式 | 核心教训 | 绑定 Skill |
|---|---|---|---|
| ce08 | 忽略网络分区，假设网络可靠 | 必须显式定义分区行为、超时与熔断 | distributed-consistency-decision |
| ce09 | 过早优化一致性牺牲可用性 | 强一致性很贵，不要强加给不需要的业务 | distributed-consistency-decision |
| ce10 | 将分布式事务当作本地事务 | 2PC 协调者是单点，优先用 Saga/幂等 | distributed-consistency-decision |
| ce11 | 信任单一数据源而不验证 | 多层存储必须配套 schema 校验与监控 | data-system-reliability-assessment |
| ce12 | 错误使用信号量导致死锁 | 悲观锁必须定义全局锁获取顺序 | concurrency-strategy-selection |
| ce14 | 认为更多线程总是更好 | 超核心数后上下文切换主导，出现反扩展 | concurrency-strategy-selection |
| ce19 | 低估并发控制开销 | Serializable/粗粒度锁会严重限制吞吐 | concurrency-strategy-selection |

## 数据与持久化

| 编号 | 失败模式 | 核心教训 | 绑定 Skill |
|---|---|---|---|
| ce13 | 忽略崩溃一致性 | 元数据写顺序错误导致文件系统损坏 | crash-recovery-persistence-design |
| ce16 | 无索引全表扫描 | 持久化设计再好也救不了 I/O 低效 | crash-recovery-persistence-design |
| ce18 | 在 OLTP 上跑复杂分析查询 | 会阻塞写操作、耗尽连接池 | data-system-reliability-assessment |
| ce26 | 在需要精确答案时用概率结构 | HyperLogLog/Count-Min Sketch 的误差不可接受 | probabilistic-data-structure-tradeoff |
| ce27 | 忽略 Bloom filter 假阳性率 | m/n/k 未按预期调整导致假阳性飙升 | probabilistic-data-structure-tradeoff |

## 抽象与建模

| 编号 | 失败模式 | 核心教训 | 绑定 Skill |
|---|---|---|---|
| ce20 | 抽象泄漏导致边界模糊 | 底层细节穿透抽象，使用者被迫处理 | abstraction-barrier-evolution |
| ce21 | 过早泛化接口 | 为假想未来需求增加复杂度 | abstraction-barrier-evolution |
| ce22 | 在应用层重复实现数据库功能 | 违反分层原则，导致一致性与性能灾难 | data-modeling-decision |

---

**完整列表**: 详见 `candidates/counter-examples.md`
