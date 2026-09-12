# 01 · 目标 JD 拆解

## 岗位画像（从 JD 提炼）

DeepSeek 推理团队 / KV Cache 方向，关键词聚类为四条主线：

1. **KV cache 全生命周期**：卸载（HBM→DRAM→NVMe→远端）、复用（前缀/跨请求/跨实例）、调度、一致性（PD 分离跨节点传输）
2. **高速 IO 数据通路**：RDMA、GPU Direct Storage、零拷贝、io_uring、SPDK、NVMe、CXL
3. **多级缓存算法**：准入、淘汰、预取、量化/压缩，提升 IO 性能与可用容量
4. **与推理引擎深度集成**：vLLM/SGLang，PD 分离下 KV 池化与全局管理；参考系：Mooncake / 3FS / FlexKV / LMCache

## 关键认知：JD 是词汇表，不是功能清单

- 技术栈是一条**瓶颈依赖链**，不是自助餐：内核拷贝开销大→零拷贝；TCP 扛不住→RDMA；CPU bounce buffer 成瓶颈→GDS；syscall 延迟主导→io_uring/SPDK；带宽比算力贵→量化。
- **每项技术都是对某个被测出的疼痛的回答**。没有疼痛就答不上"这里为什么用 SPDK 而不是 io_uring"。
- 三种合法处置方式：

| 处置 | 成本 | 产出 |
|---|---|---|
| 亲手测过并进项目 | 高（周级） | 深度证据，2~3 个就够 |
| micro-bench 测过但书面拒绝 | 低（天级） | 决策记录，面试护城河 |
| 读懂原理能深聊 30 分钟 | 最低 | 词汇流利度 |

- 唯一死刑：**声称第一种、实际第三种**。

## 叙事重构（火山实习的定位）

> "我在火山引擎做对象存储——KV Cache Store 本质就是一个 latency-critical 的分布式对象存储：块索引 = prefix radix tree，分层 = HBM/DRAM/NVMe/远端，一致性 = PD 分离下的跨节点传输。我用了几个月把这套东西在推理场景重造了一遍。"

对象存储直觉（分层、生命周期、元数据索引、压缩、QoS）全部平移，SLA 从毫秒级变微秒级。

## 必读关键数字（面试弹药）

- 带宽阶梯：HBM ~3TB/s ≫ NVLink 900GB/s > PCIe5 x16 ~64GB/s ≈ CX-7 400G ≈ 50GB/s > Gen5 NVMe ~14GB/s
- DeepSeek V3 MLA：每 token KV ≈ 70KB（576 latent × 61 层 × 2B）；1M token 上下文 ≈ 70GB
- 16-token block（MLA）≈ 1.1MB ≈ NVMe 顺序读 / RDMA 单消息效率甜区
- 3FS 最慢恢复 50–150ms，仍比重新 Prefill（16K token ≈ 3300ms）快一个数量级
- 统一 system prompt 前缀缓存命中率可达 80–95%，TTFT 降 5–10×
- FP8（H100/H200 原生）：精度损失 <1%、显存减半、提速 1.5–2×
