---
name: reliable-transport-arq
description: |
  当用户需要在不可靠介质（网络、消息队列、分布式存储、嵌入式通信）上设计可靠传输、持久化通道或请求-响应确认机制时激活。
  核心触发信号：讨论"丢包/乱序/比特错误如何恢复"、"ACK-重传-超时机制"、"滑动窗口"、"序列号设计"、"快速重传"、"动态超时计算"、"累积确认 vs 选择确认"。
  不适用于：纯信息查询（如"TCP 报文头多长"）、日常琐碎选择、应用层消息边界设计（调用 `abstract-boundary-design`）。
source_book: 《Computer Networking: A Top-Down Approach》Kurose & Ross / 《Operating Systems: Three Easy Pieces》Arpaci-Dusseau
source_chapter: CN Ch.3.4–3.5 / OSTEP Ch.36–37
tags: [reliable-transport, arq, tcp, sequence-numbers, sliding-window, timeout, fast-retransmit]
related_skills: []
---

# 不可靠信道上的可靠传输：ACK-重传-超时三元框架

## R — 原文 (Reading)

> "Fundamentally, three additional protocol capabilities are required in ARQ protocols to handle the presence of bit errors: Error detection... Receiver feedback... Retransmission. A complete and reliable data transfer protocol requires the addition of sequence numbers and timers."
>
> "TCP provides reliable data transfer by using positive acknowledgments and timers... TCP acknowledges data that has been received correctly, and it then retransmits segments when segments or their corresponding acknowledgments are thought to be lost or corrupted."
>
> — Kurose & Ross, CN Ch.3.4–3.5; Arpaci-Dusseau, OSTEP Ch.36–37

---

## I — 方法论骨架 (Interpretation)

当底层信道可能丢包、乱序或损坏比特时，可靠传输不能依赖底层网络的"善意保证"，而必须通过发送方与接收方之间的显式状态机与反馈控制来构建。核心方法论是一个五元机制：

1. **差错检测**：为每个传输单元附加校验和（checksum/CRC），使接收方能识别比特错误；
2. **接收方反馈**：接收方通过肯定确认（ACK）或否定确认（NAK）告知发送方数据状态；
3. **序列号**：为每个发送单元分配单调递增的序列号，使接收方能检测重复、丢包和乱序；
4. **定时器与重传**：发送方维护超时计时器，若超时未收到 ACK，则触发重传；
5. **性能优化**：在基础三元组之上引入累积 ACK、快速重传、动态超时估计和滑动窗口，将信道利用率从停等协议的低吞吐提升到流水线级传输。

这一框架不仅适用于 TCP，也适用于任何需要在不可靠介质上保证"发送什么、接收方就按序无差错收到什么"的系统设计。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: TCP 在不可靠 IP 网络上的可靠字节流传输
- **问题**: IP 网络不保证数据报按序到达、不保证不丢包、不保证比特无错。如何在如此不可靠的介质上为应用层提供可靠的字节流抽象？
- **方法论的使用**: TCP 将上述五元机制完整实现为一个有限状态机。三次握手同步初始序列号（ISN），防止旧连接的延迟报文干扰新连接；发送的每个 Segment 携带序列号（Seq）和校验和；接收方返回累积 ACK（Ack = 期望收到的下一个字节序号）；发送方维护重传定时器（RTO），基于 RTT 采样动态更新（EstimatedRTT + 4·DevRTT）；引入快速重传机制，收到 3 个重复 ACK 即视为丢包信号，无需等待超时；通过滑动窗口（cwnd + rwnd）实现流水线发送，最大化信道利用率。
- **结论**: 可靠性不是底层网络的馈赠，而是通过端系统上显式的状态机、反馈控制和保守探测策略构建的。
- **结果**: TCP 成为全球互联网事实上的可靠传输标准，支撑了 HTTP、SMTP、FTP 等几乎所有需要可靠交付的应用层协议。

### 案例 2: Go-Back-N 与 Selective Repeat 的协议演进
- **问题**: 停等协议（Stop-and-Wait）每发一个分组必须等待 ACK，信道利用率极低。如何在保证可靠性的前提下提高吞吐？
- **方法论的使用**: 作者展示了两种流水线协议。Go-Back-N（GBN）允许发送方连续发送窗口内 N 个未确认分组，但只维护一个定时器（对应最早未确认分组）；一旦超时，重传窗口内所有未确认分组。Selective Repeat（SR）则允许接收方缓存乱序到达的正确分组，并为每个已发送分组维护独立定时器，只重传真正丢失的分组；ACK 也变为逐个确认（或 SACK 选择确认）。
- **结论**: 窗口大小 N 的选择必须在信道利用率与缓存/状态开销之间权衡；接收方缓存能力决定了是采用"全部重传"还是"选择性重传"。
- **结果**: 现代 TCP 实际融合了 GBN 的累积 ACK 语义与 SR 的选择重传能力（SACK 选项），在实现复杂性与性能之间取得了平衡。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **设计自定义可靠传输协议**：在嵌入式系统、无人机通信、工业总线或卫星链路等场景中，标准 TCP 过于重量级或不可用，需要从零设计轻量级 ARQ 协议。
2. **优化已有传输层的性能**：现有系统在高丢包、高延迟或高带宽延迟积（BDP）网络下吞吐骤降，需要调整窗口大小、重传策略或超时计算逻辑。
3. **在消息队列/分布式存储中实现可靠投递**：例如为 Kafka、RabbitMQ 或自定义 RPC 框架设计"至少一次"或"恰好一次"语义，需要理解序列号、幂等性与去重机制。
4. **排查网络应用中的异常延迟或数据丢失**：用户报告"数据偶尔丢失""重传风暴""超时设置不合理"，需要系统性地诊断 ACK 机制、定时器参数或窗口行为。

### 语言信号 (用户的话里出现这些就应激活)

- "如何在丢包/乱序/比特错误的网络上保证可靠传输"
- "设计一个 ACK + 重传 + 超时的机制"
- "序列号应该怎么设计，窗口大小怎么选"
- "快速重传和超时重传有什么区别"
- "动态 RTO 怎么计算，Jacobson/Karn 算法怎么用"
- "累积 ACK vs 选择确认（SACK）的优劣"
- "为什么停等协议效率低，怎么改成流水线"
- "我的 RPC/消息队列需要保证消息不丢，该怎么做"

### 与相邻 skill 的区分

- 与 `tcp-congestion-control-aimd` 的区别: 本 skill 聚焦**可靠性**（丢包恢复、按序交付、无差错），而非带宽探测与公平共享。拥塞控制解决"发多少"，可靠传输解决"发了怎么确保到"。
- 与 `abstract-boundary-design` 的区别: 本 skill 不处理应用层消息边界、帧定界或协议分层抽象，而是聚焦传输层的 ACK-重传-超时状态机。
- 与 `distributed-consistency-cap-pacelc` 的区别: 本 skill 处理单条连接的端到端可靠交付，而非多副本之间的一致性模型与分区容错策略。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后，agent 应按以下步骤执行:

1. **诊断不可靠信道的具体故障模式**
   - 完成标准: 明确列出底层信道可能出现的三种故障（丢包、乱序、比特错误）及其组合概率，确认是否需要显式可靠传输机制。
   - 判停条件: 若底层信道已通过其他机制（如光纤通道的 FEC、专用硬件的可靠链路）保证可靠性，则建议无需重复构建 ARQ，跳到步骤 5 给出边界建议。

2. **选择 ARQ 基础变体（停等 / GBN / SR / 混合）**
   - 完成标准: 根据信道带宽延迟积（BDP = 带宽 × RTT）、接收方缓存限制、错误率，推荐合适的协议变体。
     - BDP 小、缓存极受限 → Stop-and-Wait
     - BDP 中等、错误率低、实现简单优先 → Go-Back-N
     - BDP 大、错误率高、接收方有缓存 → Selective Repeat 或 TCP-like 混合
   - 判停条件: 若用户已明确指定协议变体，验证其是否与约束匹配，不匹配则提出修正建议。

3. **设计序列号空间与 ACK 语义**
   - 完成标准:
     - 确定序列号位数 k，使得 2^k > 最大窗口大小（GBN 要求 2^k ≥ N+1；SR 要求 2^k ≥ 2N，防止新旧窗口重叠导致歧义）。
     - 明确 ACK 类型：累积 ACK（Ack = 下一个期望序号）或选择 ACK（SACK，列出已接收的块边界）。
     - 说明如何处理重复分组（接收方通过序列号识别并丢弃，但仍需重发 ACK，防止发送方因 ACK 丢失而无限重传）。

4. **设计定时器与重传策略**
   - 完成标准:
     - 定义超时触发条件：首次发送后启动定时器，超时未收到 ACK 则重传。
     - 给出动态 RTO 计算公式：
       - `EstimatedRTT = (1-α)·EstimatedRTT + α·SampleRTT`（典型 α = 0.125）
       - `DevRTT = (1-β)·DevRTT + β·|SampleRTT - EstimatedRTT|`（典型 β = 0.25）
       - `TimeoutInterval = EstimatedRTT + 4·DevRTT`
     - 引入 Karn/Partridge 算法：重传分组的 RTT 样本不用于更新 EstimatedRTT，避免歧义。
     - 引入快速重传：收到 3 个重复 ACK 即视为丢包信号，立即重传丢失分组，无需等待定时器超时。
     - 说明重传次数上限或超时上限，防止无限重传导致资源耗尽。

5. **设计滑动窗口与流量控制（如适用）**
   - 完成标准:
     - 定义发送窗口大小（受限于接收方通告窗口 rwnd 和拥塞窗口 cwnd，取 min(rwnd, cwnd)）。
     - 说明窗口更新机制：接收方通过 ACK 中的窗口字段告知可用缓存，发送方据此调整未确认分组数量。
     - 解释零窗口探测（Zero-Window Probe）：当接收方窗口为 0 时，发送方如何周期性探测以避免死锁。

6. **输出完整的状态机或伪代码，并给出边界警告**
   - 完成标准: 提供发送方与接收方的状态转移描述（或伪代码），标注关键事件（发送、收到 ACK、超时、收到重复分组）。
   - 同时引用 counter-examples 中的失败模式（见 B 段），提醒用户避免常见陷阱。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 底层信道本身已通过硬件 FEC、可靠链路层或专用总线保证极低误码率和按序交付，此时叠加 ARQ 属于冗余设计，可能引入不必要的延迟。
- 应用场景对实时性要求远高于可靠性（如实时音视频、在线游戏的位置同步），此时 TCP 的重传和按序交付反而会造成延迟抖动，应考虑 UDP + 前向纠错（FEC）或自适应码率，而非 ARQ。
- 问题聚焦于应用层消息边界、帧定界或协议分层抽象（如"TCP 粘包怎么解决"），应调用抽象边界设计 skill，而非本 skill。

### 作者在书中警告的失败模式

- **ce07 — 忽略传输层协议与应用层语义的不匹配**
  - 来源: CN Ch.3 Transport Layer
  - 警告: "TCP provides a reliable byte-stream abstraction, but applications often need message boundaries or timing semantics that TCP does not preserve."
  - 失败模式: 应用开发者假设 TCP 按发送边界交付数据，导致消息粘包、解析错误；或在需要实时性的场景误用 TCP，造成不必要的延迟。TCP 是字节流协议，不保留消息边界，应用层必须自行实现帧定界（长度前缀、分隔符）。

- **ce06 — 增加带宽就能解决所有延迟问题**
  - 来源: CN Ch.1 Computer Networks and the Internet
  - 警告: "Bandwidth and latency are not the same; increasing bandwidth does not reduce propagation delay."
  - 失败模式: 对于交互式应用，RTT 是主导因素；带宽升级无法减少光速传播延迟。在 ARQ 设计中，RTT 直接决定定时器下限和流水线窗口大小，单纯增加带宽而不优化 RTT 或窗口策略，对吞吐提升有限。

- **ce08 — 忽略网络分区（假设网络可靠）**
  - 来源: DDIA Ch.1 Reliable, Scalable, and Maintainable Applications
  - 警告: "Networks are not reliable. You must design for network partitions and partial failures."
  - 失败模式: 在分布式系统中，ACK 通道本身也可能丢失或延迟。若未显式处理超时、重试上限和幂等性，可能在分区时引发级联故障或数据不一致。

### 作者的盲点 / 时代局限

- 教材对现代数据中心网络中 RDMA（RoCEv2、InfiniBand）的可靠传输机制（如基于硬件的可靠连接 RC）覆盖不足，这些技术通过网卡硬件实现零拷贝、内核旁路的可靠传输，与软件 ARQ 有本质差异。
- 对 QUIC、SCTP 等多流传输协议的设计细节讨论有限。QUIC 在应用层（UDP 之上）重新实现了连接迁移、前向纠错和更快的丢包恢复，代表了 ARQ 框架在移动互联网时代的重要演进。
- 对卫星通信、深空网络等高延迟场景下的延迟容忍网络（DTN, Delay/Disruption Tolerant Networking）和 Bundle Protocol 未涉及，这些场景下的 ARQ 设计需要以"存储-转发"和" custody transfer "为核心。

### 容易混淆的邻近方法论

- **拥塞控制 (AIMD)** vs **可靠传输 (ARQ)**: 拥塞控制解决"网络能承受多少流量"，通过 cwnd 探测可用带宽；ARQ 解决"如何确保已发数据被正确接收"，通过序列号、ACK 和重传实现。两者在 TCP 中协同工作，但属于不同问题域。
- **流量控制 (Flow Control)** vs **可靠传输 (ARQ)**: 流量控制通过 rwnd 防止发送方淹没接收方缓存；ARQ 通过重传和定时器修复丢包/错误。流量控制不修复错误，ARQ 不调节速率。
- **前向纠错 (FEC)** vs **自动重传请求 (ARQ)**: FEC 通过冗余编码在接收方直接恢复少量错误，无需反馈和重传，适合实时或高延迟场景；ARQ 依赖反馈和重传，实现简单但引入延迟抖动。

---

## 相关 skills

- depends-on: layered-end-to-end-decision
- contrasts-with: (none)
- composes-with: congestion-control-aimd


## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓
- **测试通过率**: 待测 (详见 test-prompts.json)
- **蒸馏时间**: 2026/04/17
