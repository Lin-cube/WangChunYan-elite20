# WangChunYan_C5A_仓库分析

> **任务**：C5A GitHub 入门与仓库理解 —— Level 2（Silver 🥈）交付物
> **分析对象**：[neolaf2/neoskills](https://github.com/neolaf2/neoskills)（本地已克隆至 `neoskills/`）
> **作者**：WangChunYan ｜ **日期**：2026-11-22

---

## 一、这个仓库是做什么的？（一句话概括）

**neoskills 是一个面向 AI 编程智能体（Claude Code / OpenCode / OpenClaw）的“Homebrew 式”技能包管理器——让你像 brew install 一样浏览、搜索、安装、部署和管理 AI 技能（skills），并通过内置的本体图谱（ontology）管理技能之间的关系。**

---

## 二、仓库的目录结构是怎样的？

```
neoskills/
├── README.md                # 项目完整说明（安装/快速上手/架构/CLI 参考）
├── CLAUDE.md                # 面向 AI 协作者的项目指南（构建/测试/架构约定）
├── TASKS.md                 # v0.4 进度清单与设计决策日志
├── LICENSE                  # MIT 许可证
├── pyproject.toml           # Python 项目配置（依赖、CLI 入口、工具配置）
├── uv.lock                  # uv 依赖锁定文件
├── Dockerfile               # 容器化运行支持
├── main.py                  # 占位入口（打印问候语，非真正 CLI 入口）
├── neoskills_v0_1.md        # 项目设计文档 v0.1（演进历史）
├── neoskills_v0_2.md        # 项目设计文档 v0.2
├── neoskills_v0_21.md       # 项目设计文档 v0.2.1
├── neoskills_v0_3.md        # 项目设计文档 v0.3
│
├── src/neoskills/           # ★ 核心源码（约 6500 行 Python）
│   ├── cli/                 #   命令行层（Click 框架）：init/tap/list/install/
│   │                        #   create/ontology/doctor/push 等 20+ 子命令
│   ├── core/                #   核心层：Cellar（工作区管理）、config、linker
│   │                        #   （符号链接部署）、frontmatter、checksum、tap 等
│   ├── ontology/            #   本体图层（v0.4 新增）：SkillGraph 内存属性图、
│   │                        #   loader/writer、生命周期状态机、版本管理、
│   │                        #   组合/拆分、Mermaid/DOT 导出
│   ├── runtime/claude/      #   Claude Code MCP 插件（12+ 工具）
│   ├── adapters/            #   多智能体适配器（claude / opencode / openclaw）
│   ├── plugin/              #   插件上下文与 schema
│   ├── meta/                #   技能增强器（Claude 驱动的 audit/normalize）
│   └── translators/         #   翻译层（预留）
│
├── skills/                  # ★ 仓库自带的技能（详见下节）
│   ├── bank-status/         #   SKILL.md（技能库状态查询）
│   └── skill-dedup/         #   SKILL.md + scripts/dedup_scan.py（技能去重）
│
├── agents/                  # ★ 自主智能体定义（4 个，供 Claude Code Task 调用）
│   ├── skill-scanner.md     #   扫描发现各智能体生态中的技能
│   ├── skill-importer.md    #   技能导入（target/git/URL → 技能库）
│   ├── skill-deployer.md    #   技能部署（符号链接/变体选择/备份恢复）
│   └── skill-dedup.md       #   重复技能识别与消解
│
├── commands/
│   └── ns.md                # /ns 斜杠命令（会话内调用 neoskills CLI/agents）
│
├── tests/                   # 测试（172 个用例）
│   ├── unit/                #   单元测试 12 个模块（core/ontology/cli/linker…）
│   └── integration/         #   端到端集成测试
│
├── docs/
│   ├── ontology-design.md   # 本体层完整设计文档
│   └── plans/               # 技能组织功能的设计与实现计划
│
└── .github/workflows/       # CI（ci.yml）与发布（release.yml）流水线
```

---

## 三、里面有哪些技能（skills）？各自做什么？

仓库里有 **2 个正式技能**（`skills/` 目录）+ **4 个智能体定义**（`agents/` 目录）+ **1 个斜杠命令**（`commands/`），共同构成技能管理闭环：

### 正式技能（skills/）

| 技能 | 触发场景 | 做什么 |
|------|---------|--------|
| **bank-status** | 用户询问技能库状态、库存、有哪些技能可用 | 读取 `~/.neoskills/` 工作区的注册表与状态文件，汇总输出：技能总数、目标（targets）配置、当前已嵌入（符号链接）的技能、最近 5 次导入记录；并给出后续操作建议（scan / import / embed） |
| **skill-dedup** | 用户提到“重复技能”、导入后清理、审计技能库存 | 调用自带脚本 `dedup_scan.py` 扫描技能库与各智能体目标目录，把重复分为三类：**完全重复**（SHA256 相同，可安全合并为符号链接）、**分叉副本**（同名不同内容，推荐更丰富的版本入库）、**名称相似组**（需人工判断）；支持 `--resolve exact/diverged/all` 自动消解和 `--dry-run` 预览，操作前自动备份 |

### 自主智能体（agents/，可多步自主工作）

| 智能体 | 职责 |
|--------|------|
| **skill-scanner** | 自主发现：遍历各智能体生态的目录，解析 SKILL.md frontmatter，产出技能清单 |
| **skill-importer** | 自主导入：从目标目录 / Git 仓库 / 网页 URL 抓取技能，分析后放入技能库（bank） |
| **skill-deployer** | 自主部署：把技能/技能包部署到目标智能体，处理符号链接创建、备份、恢复 |
| **skill-dedup** | 自主去重：识别并消解跨库重复技能（与上面的 skill-dedup 技能配套） |

### 会话命令（commands/）

| 命令 | 职责 |
|------|------|
| **/ns** | 在 Claude Code 会话内直接调用 neoskills：简单命令走 CLI（validate/install/embed），复杂命令路由给上面 4 个智能体（scan/import/deploy/dedup） |

---

## 四、你觉得哪个技能最有意思？为什么？

**我选 skill-dedup（技能去重）。**

理由：

1. **它解决的是真实且隐蔽的痛点。** 用 AI 编程智能体的人往往在多个地方（Claude Code、OpenCode、插件缓存、不同仓库）积累大量技能，时间一长必然产生“同名不同内容”和“内容相同多处存放”的混乱。这个问题平时感觉不到，一旦技能多了就非常头疼——它属于“基础设施型”技能。
2. **设计有层次感。** 它没有做简单粗暴的“同名即删”，而是把重复细分为完全重复 / 分叉副本 / 名称相似三类，分别对应“自动合并”“择优入库”“人工裁决”三种策略——这体现了一种对数据安全的敬畏：能自动的自动，不能自动的绝不乱动。
3. **工程细节到位。** 脚本支持 `--dry-run` 预览、自动备份（`.dedup-backup`）、JSON 输出供程序消费、相似度阈值可调。这些细节让一个“清理类”工具用起来足够放心。
4. **它与本仓库的核心理念互相呼应。** neoskills 的部署模型是“单一事实来源 + 符号链接零拷贝”，而 skill-dedup 正是把游离在体系外的重复副本“收编”回这个体系的清道夫——不理解部署模型，就看不懂它为什么这么设计，这让它成为理解整个仓库的一把钥匙。

---

## 五、补充观察（我的个人思考）

- **本体图层（ontology）是仓库的“野心”所在**：技能不只是文档，而是图谱中的节点，带类型化的关系边（requires / extends / composes_with / conflicts_with…）、生命周期状态机（candidate → … → archived）和语义化版本。这让“技能管理”升级为“技能资产管理”。
- **渐进式富化（L0–L3）的设计很务实**：不强制所有技能一开始就有完整元数据，裸 SKILL.md（L0）就能用，元数据逐步补齐——这是对“存量技能兼容”的务实妥协，值得学习。
- **一个小瑕疵**：根目录 `main.py` 只是一个打印 "Hello from neoskills!" 的占位文件，真正的 CLI 入口是 `pyproject.toml` 里声明的 `neoskills.cli.main:cli`，新读者可能会短暂困惑。

---

*本文档为 Level 2 交付物。更深入的架构分析见 `WangChunYan_C5A_深度分析报告.md`（Level 4）。*
