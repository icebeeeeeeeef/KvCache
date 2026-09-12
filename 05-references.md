# 05 · 参考文献与案例库

## 论文（按精读优先级）

| 论文 | 学什么 |
|---|---|
| PagedAttention (vLLM) | 块化 KV 管理的原点 |
| Mooncake (Kimi) | 舰队级 KV 池化 + 传输优先架构 + **开源生产 trace** |
| DistServe / Splitwise | PD 分离开山 |
| CacheGen | KV 压缩与流式传输 |
| KIVI / KVQuant | KV 量化的精度-容量权衡 |
| FlexGen | offloading 鼻祖 |

## 系统（三个假设不同的标本）

| 系统 | 编码的假设 | 读它学什么 |
|---|---|---|
| **Mooncake** | 月之暗面自家舰队、Kimi 负载、多 NIC 400G；传输优先，cache 语义薄 | Transfer Engine；生产 trace |
| **LMCache** | 学术界（UChicago 系）、vLLM 生态嵌入优先 | vLLM KVConnector 接入姿势；引擎侧视角 |
| **PegaFlow** (novitalabs) | 生命周期解耦优先（sidecar 进程边界）；Rust 数据面 | goals/non-goals 纪律；单变量实验设计；HLL 上限估算 |
| **3FS** (DeepSeek) | 训练 checkpoint 与 KV 共用文件系统 | chunk engine + USRBIO = RDMA+io_uring+零拷贝教科书；**读它 = 读面试官的品味** |
| NIXL (NVIDIA) | 传输抽象层，不做缓存语义 | 传输层 API 设计 |
| SGLang HiCache | 引擎内分层缓存 | 另一个集成面的参考 |

**共存逻辑**：系统无好坏，只有假设。"哪个最强"是外行问题，"它编码了什么假设、假设何时失效"是内行话语。

## 公开 trace（轨道 A 的弹药）

- Mooncake 生产 trace（开源）
- Azure LLM inference trace
- ShareGPT / LMSYS-Chat-1M / BurstGPT

## 社区接入点（PR 优先级）

1. **3FS**（DeepSeek 亲儿子，merged PR = 简历直递内网）
2. vLLM（KVConnector 生态）
3. LMCache / Mooncake / PegaFlow

贡献质量标准：**一份带完整复现脚本+环境+火焰图的性能 issue > 十个 typo PR**。

## 行业观察要点（来自运维视角杂谈文章）

- RoCEv2 占比上升，但 PFC/ECN/DCQCN 无损配置是真实痛点（可做的实验：故意配错测尾延迟劣化）
- PD 分离不是万能药：网络不达标反而增加延迟 → 调度器需"负决策"能力（知道何时不缓存）
- H200 141GB HBM 是卸载的反作用力 → 威胁分析一节：价值锚在长上下文+agentic+高前缀复用
- 推理成本战争化：单 token 降 30% = 日省数十万 → benchmark 报告必须有 $/M tokens 一节
- 双生态（NVIDIA/昇腾）长期共存 → 传输后端可插拔写进设计文档
- 企业宁挖成熟工程师不培养新人 → 公开 repo+数据+踩坑实录 = "免培养证明"

## 灵感公式

> "我在复现 X 时测到 Y 与预期不符，追查后发现 Z。这个项目就是为搞清楚 Z 而生的。"
