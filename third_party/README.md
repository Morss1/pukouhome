# Third-party Projects

本目录用于管理本项目依赖、研究或参与开发的外部开源项目。

这些项目通常以 Git Submodule 的方式引入，保留各自独立的 Git 历史。本仓库只记录 Submodule 当前指向的 Commit；对外部项目的实际修改应提交到对应的 Fork 仓库。

## CodeGraph

目录：[`codegraph/`](./codegraph/)

上游仓库：[colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)

个人 Fork：[Morss1/codegraph](https://github.com/Morss1/codegraph)

CodeGraph 是一个本地代码知识图谱工具，可以索引项目中的符号、调用关系和文件结构，为 Codex 等编程 Agent 提供更高效的代码检索与理解能力。

### 引入目的

目前主要将 CodeGraph 用于：

- 辅助 Codex 理解和检索大型代码仓库。
- 学习 CodeGraph 的代码索引、语言解析和关系提取机制。
- 基于个人 Fork 尝试功能开发并参与上游贡献。
- 为 CUDA 项目补充 `.cu` 和 `.cuh` 文件的索引支持。

### CUDA 支持计划

CodeGraph 当前支持 C/C++，但尚未正式支持 CUDA 源文件扩展名 `.cu` 和 `.cuh`。

CUDA 支持需求最初记录在上游 Issue [#387](https://github.com/colbymchenry/codegraph/issues/387)，随后被维护者收录到统一的语言支持路线图 [#648](https://github.com/colbymchenry/codegraph/issues/648)。

维护者指出，CodeGraph 基于 Tree-sitter，这项工作可以从复用现有 C/C++ Extractor 开始。第一阶段计划尝试：

1. 将 `.cu` 和 `.cuh` 文件识别为 C++ 方言。
2. 使用现有 C/C++ Extractor 提取大部分函数、类型和调用关系。
3. 添加 CUDA 文件识别与索引测试。
4. 验证普通 C++、CUDA Kernel 和 Host 端代码的提取结果。
5. 进一步评估 `__global__`、`__device__`、`__host__` 和 `<<<...>>>` Kernel Launch 语法。
6. 功能稳定后，尝试向上游提交 Pull Request。

### 仓库关系

```text
colbymchenry/codegraph
        │
        │ upstream
        ▼
Morss1/codegraph
        │
        │ Git Submodule
        ▼
pukouhome/third_party/codegraph
```

- `upstream` 用于同步 CodeGraph 官方仓库。
- `origin` 指向个人 Fork，用于保存开发分支和提交。
- `pukouhome` 只保存 Submodule 的 Commit 引用。

### 开发流程

进入 CodeGraph Submodule：

```bash
cd third_party/codegraph
```

同步上游：

```bash
git fetch upstream
git switch main
git rebase upstream/main
git push origin main
```

创建 CUDA 支持开发分支：

```bash
git switch -c feat/cuda-language-support
```

完成修改后，先在 CodeGraph Fork 中提交并推送：

```bash
git add .
git commit -m "feat: add CUDA source file support"
git push -u origin feat/cuda-language-support
```

然后回到 `pukouhome` 更新 Submodule 引用：

```bash
cd ../..
git add third_party/codegraph
git commit -m "chore: update codegraph submodule"
```

### 克隆方式

克隆本仓库时同时初始化 Submodule：

```bash
git clone --recurse-submodules <repository-url>
```

如果已经完成普通克隆：

```bash
git submodule update --init --recursive
```

