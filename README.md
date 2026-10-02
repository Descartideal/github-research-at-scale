# GitHub Research at Scale

一个用于大规模 GitHub 资料搜集的通用 Agent Skill：先跨仓搜索和批量筛选，再深读少量候选，减少逐仓库 GitHub API 请求。

An Agent Skill for large GitHub surveys: discover once, reuse metadata, search across repositories, and inspect a bounded shortlist.

## 解决什么问题

当一次任务涉及几百或上千个仓库，逐个读取 README、文件树、代码和元数据会产生大量请求。这个 Skill 指导 Agent 按问题选择 GitHub、Sourcegraph / Zoekt 等索引、现有缓存和本地 Git。

```text
全局发现 → 保存并去重候选 → 元数据 / 跨仓代码筛选 → shortlist → 文件证据 / 本地深读
```

它是一组行为指令，不是 MCP 服务或限流代理。不会自动安装 Sourcegraph，也不能强制保证 Agent 的请求数。没有索引服务时，会缩小候选范围并说明覆盖不足，而不是默认逐个扫描上千仓库。

## 安装

克隆本公开仓库，再把 `skills/github-research-at-scale` 文件夹复制到 Agent 支持的 skills 目录。目录中必须直接包含 `SKILL.md`。

Codex 用户级安装示例（PowerShell；目标已存在时先检查，避免覆盖自己的修改）：

```powershell
git clone https://github.com/Descartideal/github-research-at-scale.git
$skillTarget = Join-Path $env:USERPROFILE '.codex/skills/github-research-at-scale'
if (Test-Path -LiteralPath $skillTarget) { throw 'Skill 已存在，请先检查现有版本。' }
Copy-Item -LiteralPath './github-research-at-scale/skills/github-research-at-scale' -Destination $skillTarget -Recurse
```

兼容 `SKILL.md` 的其他 Agent 可按其当前文档使用项目级或用户级 skills 目录，例如 `.agents/skills`。DSH 的具体目录、发现方式和 MCP 配置应以你使用的版本文档为准。`agents/openai.yaml` 是可选的 Codex UI 元数据。

## 使用示例

```text
使用 $github-research-at-scale 搜集实现多 Agent 协作的 TypeScript 开源项目。
先全局发现并复用元数据，再根据代码证据筛出 15 个候选。
对最相关的 5 个做深读，说明检索范围、未覆盖区域和文件证据。
```

```text
使用 $github-research-at-scale 在大量仓库中找 registerTool 的实现。
优先使用已连接的跨仓代码索引；如果没有索引，给出有限范围的结果，
不要逐仓库调用 GitHub code search。
```

自动发现由 Agent 决定；在 Codex 中也可通过 `$github-research-at-scale` 显式调用。其他 Agent 的显式调用语法以自身文档为准。

## 设计要点

- 全局发现只做一次；如果已有获授权的多个 Agent，共享候选并分工深读。
- 已返回的元数据直接复用；只获取决策需要的缺失字段。
- 索引覆盖不足、超时、截断及 GitHub 查询上限需要明确记录。
- 限流根据实际诊断处理，不盲目重试，也不把所有 403 当限流。
- Sourcegraph / Zoekt 是可选搜索后端；按当前工具能力选择，不假设固定 MCP 工具名。
- 不声称采样结果穷尽 GitHub，也不承诺未经测量的配额节省比例。

## 项目结构

```text
skills/github-research-at-scale/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── indexed-search.md
    └── github-limits.md
```

核心指令使用英文以便不同 Agent 复用；本说明提供中文安装和使用介绍。不需要 Python 或 Node 运行时，只有本地深读时需要 Git / ripgrep 等相应工具。

## 检查与贡献

参见 [CHECKS.md](CHECKS.md) 中的结构检查与行为场景。修改后检查路由是否符合真实工具能力、预算和用户目的。结构校验不能证明实际模型每次都会遵循这些指令。

MIT License。项目整理自用户提出的大规模 GitHub 调研思路，未附带原始聊天记录、账号凭据或私有研究数据。
