# 项目 Skills 说明

本目录中的 Skill 来源于另一台机器的 Codex Home，原始路径为 `.codex/skills/`。导入本项目时，它们被放置到 `.agents/skills/`，作为项目级 Skill 使用。

这些 Skill 并不都是完全独立的，其中一些是总控 Skill，负责组合和调用多个子 Skill 完成一套完整工作流。

## 文件结构

### Skill 的常见内部结构

```text
<skill-name>/
├── SKILL.md          # 必需：Skill 的说明、触发条件和执行流程
├── scripts/          # 可选：Skill 使用的脚本和工具
├── references/       # 可选：知识库、参考文档和数据
├── assets/           # 可选：模板、图片或其他静态资源
├── agents/           # 可选：Agent 相关配置
└── configs/          # 可选：运行配置、模型配置或测试配置
```

其中 `SKILL.md` 是每个 Skill 的入口文件。Codex 会根据其中的 `name`、`description` 和正文说明，判断何时以及如何使用该 Skill。

## Skill 关系概览

```text
CUDA Kernel 优化
└── cuda-optimizer
    ├── cuda-code-generator
    ├── kernel-benchmarker
    └── ncu-rep-analyzer

SGLang SOTA 优化
└── sglang-sota-humanize-loop
    ├── llm-serving-auto-benchmark
    ├── llm-torch-profiler-analysis
    ├── llm-pipeline-analysis
    └── model-pr-history-knowledge

vLLM SOTA 优化
└── vllm-sota-humanize-loop
    ├── llm-serving-auto-benchmark
    ├── llm-torch-profiler-analysis
    ├── llm-pipeline-analysis
    └── model-pr-history-knowledge
```

## CUDA Kernel 优化工作流

### `cuda-optimizer`

CUDA Kernel 优化流程的总控 Skill。

它负责持续执行“生成或修改代码 → 正确性验证 → 性能测试 → NCU 分析 → 再次优化”的循环，直到性能收敛或没有明显优化空间。

它会组合以下三个子 Skill：

| 子 Skill | 作用 |
| --- | --- |
| `cuda-code-generator` | 根据算法需求或性能分析报告生成、修复和优化 CUDA Kernel |
| `kernel-benchmarker` | 编译 Kernel，并进行正确性验证与性能测试 |
| `ncu-rep-analyzer` | 使用 NCU 采集性能数据、分析瓶颈并生成优化建议 |

### `cuda-knowledge`

CUDA 开发知识库，覆盖 Kernel 调试、性能分析、CUDA 工具、cuBLAS、NCCL、多 GPU 通信和常见优化方法。

它不是 `cuda-optimizer` 的固定子流程，但可以为 CUDA 开发、故障排查和优化判断提供参考。

### `cuda-samples`

整理 NVIDIA 官方 CUDA Samples 中的常见代码模式，包括 Reduction、GEMM、Tensor Core、CUDA Graph、多流并发和多 GPU 等示例。

适合在实现 Kernel 或查找官方代码范式时使用。

## LLM Serving 与 SOTA 优化工作流

### `sglang-sota-humanize-loop`

SGLang 模型级性能优化的总控 Skill。

它先在相同模型、硬件、请求负载和 SLA 下比较 SGLang、vLLM 与 TensorRT-LLM，然后根据性能差距循环执行 Profiling、分层分析、源码修改和重新验证，目标是让 SGLang 达到或超过最佳基线。

### `vllm-sota-humanize-loop`

与 `sglang-sota-humanize-loop` 的整体结构相同，但优化目标是 vLLM。

### 两个 SOTA 总控 Skill 的主要依赖

| Skill | 作用 |
| --- | --- |
| `llm-serving-auto-benchmark` | 在统一条件下比较 SGLang、vLLM、TensorRT-LLM 等服务框架 |
| `llm-torch-profiler-analysis` | 分析 LLM 服务框架生成的 PyTorch Profiler Trace |
| `llm-pipeline-analysis` | 从前向过程、Layer 和 Kernel 级别进一步分析性能时间线 |
| `model-pr-history-knowledge` | 查询 SGLang 和 vLLM 中不同模型家族的历史优化 PR 与实现经验 |

### 相关辅助 Skill

| Skill | 作用 |
| --- | --- |
| `llm-serving-capacity-planner` | 根据启动日志分析显存分配、KV Cache 和并发容量 |
| `model-compute-simulation` | 根据模型结构估算计算量、Tensor Shape 和 MFU |
| `model-architecture-diagram` | 生成模型架构图或结构说明 |
| `sglang-humanize-review` | 参考 SGLang 维护者历史 Review 风格进行代码审查 |
| `sglang-prod-incident-triage` | 对 SGLang 线上故障、延迟回退、崩溃和分布式卡死进行排查 |

## 独立 Skill

### `codex-token-usage`

读取本地 Codex 会话日志，统计指定时间范围内的输入、输出、缓存和推理 Token 使用量。

### `pua`

在任务反复失败或即将过早放弃时触发，要求继续排查、验证假设并尝试其他可行方案。

它属于通用行为增强 Skill，不对应某个特定技术领域。

## 当前已知的外部依赖

`sglang-sota-humanize-loop` 和 `vllm-sota-humanize-loop` 的说明中还引用了以下能力：

- Humanize Runtime
- `ncu-report-skill`
- 与实际 GPU、容器或远程机器对应的 Host/Operator Skill

这些内容不在当前导入的压缩包中。因此，本目录保留了两个 SOTA 总控 Skill 的主体及主要分析组件，但还不能认为是原机器完整运行环境的一比一备份。

实际运行相关工作流前，应先检查所引用的外部 Skill、脚本、GPU 环境和服务框架是否可用。

## 使用建议

- 需要完整优化 CUDA Kernel 时，优先使用 `cuda-optimizer`，而不是分别手动调用其子 Skill。
- 只需要生成 Kernel、性能测试或 NCU 分析时，可以直接使用对应的单一子 Skill。
- 需要优化完整的 SGLang 或 vLLM 模型服务性能时，使用对应的 SOTA 总控 Skill。
- 只需要分析已有 Trace、容量日志或历史 PR 时，直接使用对应的分析 Skill。
