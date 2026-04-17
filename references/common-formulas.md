# 跨 Skill 通用公式速查

## 性能分析

**CPU Time**
```
CPU Time = Instruction Count × CPI × Clock Cycle Time
```
> 来源: CAQA | Skill: quantitative-performance-analysis

**Amdahl's Law — 加速上限**
```
Speedup_overall = 1 / [(1 - f) + f/s]
```
其中 f = 可加速部分占比，s = 该部分加速比。
> 来源: CAQA | Skill: quantitative-performance-analysis

**性能-功耗关系**
```
P = C × V² × f
```
> 来源: CAQA | Skill: quantitative-performance-analysis

---

## Bloom Filter

**最优哈希函数数**
```
k = (m/n) × ln 2
```

**假阳性率**
```
p ≈ (1/2)^k ≈ (0.6185)^(m/n)
```
其中 m = 位数组长度，n = 预期元素数。
> 来源: MCS / AADS | Skill: probabilistic-data-structure-tradeoff

---

## 生日悖论 / 哈希碰撞

**碰撞概率**
```
P(collision) ≈ 1 - e^(-k(k-1)/2m)
```
当 k ≈ 1.177√m 时，碰撞概率超过 50%。
> 来源: MCS | Skill: probabilistic-data-structure-tradeoff

---

## AIMD 拥塞控制

**加性增长**
```
cwnd ← cwnd + MSS × (MSS/cwnd)   [每 ACK]
```

**乘性减小**
```
cwnd ← cwnd / 2                  [丢包后]
ssthresh ← cwnd / 2
```
> 来源: CN | Skill: congestion-control-aimd

---

## 期望线性性

```
E[X + Y] = E[X] + E[Y]       （即使 X 与 Y 不独立）
```
> 来源: MCS | Skills: probabilistic-data-structure-tradeoff, formal-proof-invariant-verification
