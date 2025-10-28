<div align="center">
    <img src="./media/logo_small.webp" alt="Spec Kit 标志"/>
    <h1>🌱 Spec Kit</h1>
    <h3><em>更快地构建高质量软件。</em></h3>
</div>

<p align="center">
    <strong>一个开源工具包，让你将精力放在产品场景与可预期的结果上，而不是从零开始凭感觉写每一行代码。</strong>
</p>

<p align="center">
    <a href="https://github.com/github/spec-kit/actions/workflows/release.yml"><img src="https://github.com/github/spec-kit/actions/workflows/release.yml/badge.svg" alt="Release"/></a>
    <a href="https://github.com/github/spec-kit/stargazers"><img src="https://img.shields.io/github/stars/github/spec-kit?style=social" alt="GitHub stars"/></a>
    <a href="https://github.com/github/spec-kit/blob/main/LICENSE"><img src="https://img.shields.io/github/license/github/spec-kit" alt="License"/></a>
    <a href="https://github.github.io/spec-kit/"><img src="https://img.shields.io/badge/docs-GitHub_Pages-blue" alt="Documentation"/></a>
</p>

<p align="center">
  <a href="./README.md">English</a> | 简体中文
</p>

---

## 目录

- [🤔 什么是规格驱动开发（Spec-Driven Development）？](#-what-is-spec-driven-development)
- [⚡ 快速开始](#-get-started)
- [📽️ 视频概览](#️-video-overview)
- [🤖 支持的 AI 代理](#-supported-ai-agents)
- [🔧 Specify CLI 参考](#-specify-cli-reference)
- [📚 核心理念](#-core-philosophy)
- [🌟 开发阶段](#-development-phases)
- [🎯 实验目标](#-experimental-goals)
- [🔧 前置条件](#-prerequisites)
- [📖 进一步了解](#-learn-more)
- [📋 详细流程](#-detailed-process)
- [🔍 故障排查](#-troubleshooting)
- [👥 维护者](#-maintainers)
- [💬 支持](#-support)
- [🙏 致谢](#-acknowledgements)
- [📄 许可证](#-license)

<a id="-what-is-spec-driven-development"></a>

## 🤔 什么是规格驱动开发？

规格驱动开发（Spec-Driven Development）从根本上改变了传统的软件开发方式。几十年来，代码一直是“王者”——规格通常只是辅助搭建的脚手架，一旦进入“真正的编码工作”就被抛在一边。规格驱动开发改变了这一点：规格本身变为“可执行”的，能够直接生成可工作的实现，而不只是给出指导。

<a id="-get-started"></a>

## ⚡ 快速开始

### 1. 安装 Specify CLI

选择你喜欢的安装方式：

#### 方式一：持久安装（推荐）

一次安装，处处可用：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

之后可直接使用：

```bash
specify init <PROJECT_NAME>
specify check
```

升级 Specify：

```bash
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git
```

#### 方式二：一次性使用

无需安装，直接运行：

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init <PROJECT_NAME>
```

持久安装的优势：

- 工具常驻 PATH，可随时调用
- 无需维护 shell 别名
- 通过 `uv tool list`、`uv tool upgrade`、`uv tool uninstall` 更好地管理工具
- 更干净的 shell 配置

### 2. 确立项目原则

在项目目录中启动你的 AI 助手。助手机器人中已内置 `/speckit.*` 命令。

使用 **`/speckit.constitution`** 命令来创建项目的治理原则与开发指南，它将指导后续所有开发。

```bash
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements
```

### 3. 创建规格（Spec）

使用 **`/speckit.specify`** 命令描述你要构建的内容。聚焦于“做什么”和“为什么”，暂时不要关心技术栈。

```bash
/speckit.specify Build an application that can help me organize my photos in separate photo albums. Albums are grouped by date and can be re-organized by dragging and dropping on the main page. Albums are never in other nested albums. Within each album, photos are previewed in a tile-like interface.
```

### 4. 制定技术实现计划

使用 **`/speckit.plan`** 命令提供你的技术栈与架构选择。

```bash
/speckit.plan The application uses Vite with minimal number of libraries. Use vanilla HTML, CSS, and JavaScript as much as possible. Images are not uploaded anywhere and metadata is stored in a local SQLite database.
```

### 5. 拆分任务

使用 **`/speckit.tasks`** 根据实现计划生成可执行的任务清单。

```bash
/speckit.tasks
```

### 6. 执行实现

使用 **`/speckit.implement`** 来按计划执行任务，构建你的功能。

```bash
/speckit.implement
```

更多分步说明请查看我们的[完整指南](./spec-driven.md)。

<a id="️-video-overview"></a>

## 📽️ 视频概览

想快速了解 Spec Kit？观看我们的[视频介绍](https://www.youtube.com/watch?v=a9eR1xsfvHg&pp=0gcJCckJAYcqIYzv)！

[![Spec Kit video header](/media/spec-kit-video-header.jpg)](https://www.youtube.com/watch?v=a9eR1xsfvHg&pp=0gcJCckJAYcqIYzv)

<a id="-supported-ai-agents"></a>

## 🤖 支持的 AI 代理

| 代理 | 支持 | 备注 |
|------|------|------|
| [Claude Code](https://www.anthropic.com/claude-code) | ✅ | |
| [GitHub Copilot](https://code.visualstudio.com/) | ✅ | |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | ✅ | |
| [Cursor](https://cursor.sh/) | ✅ | |
| [Qwen Code](https://github.com/QwenLM/qwen-code) | ✅ | |
| [opencode](https://opencode.ai/) | ✅ | |
| [Windsurf](https://windsurf.com/) | ✅ | |
| [Kilo Code](https://github.com/Kilo-Org/kilocode) | ✅ | |
| [Auggie CLI](https://docs.augmentcode.com/cli/overview) | ✅ | |
| [CodeBuddy CLI](https://www.codebuddy.ai/cli) | ✅ | |
| [Roo Code](https://roocode.com/) | ✅ | |
| [Codex CLI](https://github.com/openai/codex) | ✅ | |
| [Amazon Q Developer CLI](https://aws.amazon.com/developer/learning/q-developer-cli/) | ⚠️ | Amazon Q Developer CLI [暂不支持](https://github.com/aws/amazon-q-developer-cli/issues/3064) 为斜杠命令传入自定义参数。 |
| [Amp](https://ampcode.com/) | ✅ | |

<a id="-specify-cli-reference"></a>

## 🔧 Specify CLI 参考

`specify` 命令支持以下选项：

### 子命令

| 命令 | 描述 |
|------|------|
| `init` | 从最新模板初始化一个新的 Specify 项目 |
| `check` | 检查已安装工具（`git`、`claude`、`gemini`、`code`/`code-insiders`、`cursor-agent`、`windsurf`、`qwen`、`opencode`、`codex`）|

### `specify init` 参数与选项

| 参数/选项 | 类型 | 描述 |
|-----------|------|------|
| `<project-name>` | 参数 | 新项目目录名称（若使用 `--here` 可选，或使用 `.` 表示当前目录） |
| `--ai` | 选项 | 指定 AI 助手：`claude`、`gemini`、`copilot`、`cursor-agent`、`qwen`、`opencode`、`codex`、`windsurf`、`kilocode`、`auggie`、`roo`、`codebuddy`、`amp`、`q` |
| `--script` | 选项 | 选择脚本类型：`sh`（bash/zsh）或 `ps`（PowerShell） |
| `--ignore-agent-tools` | 标志 | 跳过对 AI 代理工具（如 Claude Code）的检查 |
| `--no-git` | 标志 | 跳过 Git 仓库初始化 |
| `--here` | 标志 | 在当前目录初始化项目，而不是创建新目录 |
| `--force` | 标志 | 在当前目录初始化时强制合并/覆盖（跳过确认） |
| `--skip-tls` | 标志 | 跳过 SSL/TLS 校验（不推荐） |
| `--debug` | 标志 | 启用详细的调试输出，便于排障 |
| `--github-token` | 选项 | GitHub API 令牌（或设置环境变量 GH_TOKEN/GITHUB_TOKEN） |

### 示例

```bash
# 基本初始化
specify init my-project

# 指定 AI 助手
specify init my-project --ai claude

# 使用 Cursor 支持
specify init my-project --ai cursor-agent

# 使用 Windsurf 支持
specify init my-project --ai windsurf

# 使用 Amp 支持
specify init my-project --ai amp

# 使用 PowerShell 脚本（Windows/跨平台）
specify init my-project --ai copilot --script ps

# 在当前目录初始化
specify init . --ai copilot
# 或者使用 --here 标志
specify init --here --ai copilot

# 在当前（非空）目录强制合并且不询问
specify init . --force --ai copilot
# 或者
specify init --here --force --ai copilot

# 跳过 Git 初始化
specify init my-project --ai gemini --no-git

# 启用调试输出
specify init my-project --ai claude --debug

# 为 API 请求提供 GitHub 令牌（企业环境常用）
specify init my-project --ai claude --github-token ghp_your_token_here

# 检查系统要求
specify check
```

### 可用的斜杠命令（Slash Commands）

运行 `specify init` 后，你的 AI 编码代理可以使用以下斜杠命令来进行结构化开发：

#### 核心命令

Spec-Driven Development 工作流的基础命令：

| 命令 | 描述 |
|------|------|
| `/speckit.constitution` | 创建或更新项目治理原则与开发指南 |
| `/speckit.specify` | 定义你要构建的内容（需求与用户故事） |
| `/speckit.plan` | 使用选定技术栈创建技术实现计划 |
| `/speckit.tasks` | 生成可执行的任务清单 |
| `/speckit.implement` | 按计划执行所有任务并实现功能 |

#### 可选命令

提升质量与验证的辅助命令：

| 命令 | 描述 |
|------|------|
| `/speckit.clarify` | 澄清规格中的不明确部分（推荐在 `/speckit.plan` 之前运行；原名 `/quizme`） |
| `/speckit.analyze` | 跨工件一致性与覆盖分析（在 `/speckit.tasks` 之后、`/speckit.implement` 之前运行） |
| `/speckit.checklist` | 生成自定义质量检查清单，验证需求的完整性、清晰性与一致性（如“英语单元测试”） |

### 环境变量

| 变量 | 描述 |
|------|------|
| `SPECIFY_FEATURE` | 为非 Git 仓库覆盖功能检测。设置为功能目录名（例如 `001-photo-albums`），用于在不使用 Git 分支时在特定功能上工作。<br/>**必须在使用 `/speckit.plan` 或后续命令之前，在你所使用的代理上下文中设置。** |

<a id="-core-philosophy"></a>

## 📚 核心理念

规格驱动开发（SDD）强调：

- 以“意图驱动”的方式先定义“做什么”，再决定“怎么做”
- 借助护栏与组织原则产出“高质量规格”
- 通过多步精炼，而非一次性提示生成所有代码
- 充分依赖先进 AI 模型对规格的解释与执行能力

<a id="-development-phases"></a>

## 🌟 开发阶段

| 阶段 | 关注点 | 关键活动 |
|------|--------|----------|
| **0 到 1 开发**（全新） | 从零生成 | <ul><li>从高层需求开始</li><li>生成规格</li><li>制定实现步骤</li><li>构建生产级应用</li></ul> |
| **创意探索** | 并行实现 | <ul><li>探索多样化解决方案</li><li>支持多种技术栈与架构</li><li>尝试多种 UX 模式</li></ul> |
| **迭代增强**（存量/改造） | 现代化现有系统 | <ul><li>迭代添加功能</li><li>现代化遗留系统</li><li>适配流程与规范</li></ul> |

<a id="-experimental-goals"></a>

## 🎯 实验目标

我们的研究与实验聚焦于：

### 技术中立

- 使用多样技术栈创建应用
- 验证“规格驱动开发是与技术、语言、框架无关的过程”的假设

### 企业级约束

- 演示关键业务系统的开发能力
- 纳入组织约束（云厂商、技术栈、工程实践）
- 支持企业设计系统与合规要求

### 以用户为中心

- 面向不同用户群体与偏好构建应用
- 兼容多种开发方式（从传统到 AI 原生）

### 创意与迭代

- 验证“并行实现探索”的可行性
- 提供稳健的迭代式特性开发工作流
- 扩展到升级与现代化改造场景

<a id="-prerequisites"></a>

## 🔧 前置条件

- **Linux/macOS/Windows**
- 已安装且[受支持](#-supported-ai-agents)的 AI 编码代理
- [uv](https://docs.astral.sh/uv/) 包管理器
- [Python 3.11+](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)

如果你在使用某个代理时遇到问题，请提交 Issue，以便我们改进集成。

<a id="-learn-more"></a>

## 📖 进一步了解

- **[规格驱动开发全流程](./spec-driven.md)** —— 深入了解完整方法论
- **[详细演练](#-detailed-process)** —— 逐步实现指南

---

<a id="-detailed-process"></a>

## 📋 详细流程

<details>
<summary>点击展开分步演练</summary>

你可以使用 Specify CLI 引导初始化项目，它会将所需工件带入你的本地环境。运行：

```bash
specify init <project_name>
```

或者在当前目录初始化：

```bash
specify init .
# 或使用 --here 标志
specify init --here
# 当目录中已有文件时跳过确认
specify init . --force
# 或者
specify init --here --force
```

![在终端中使用 Specify CLI 启动新项目](./media/specify_cli.gif)

系统会提示你选择使用的 AI 代理。你也可以在终端中直接指定：

```bash
specify init <project_name> --ai claude
specify init <project_name> --ai gemini
specify init <project_name> --ai copilot

# 或者在当前目录：
specify init . --ai claude
specify init . --ai codex

# 或使用 --here 标志
specify init --here --ai claude
specify init --here --ai codex

# 在非空当前目录强制合并
specify init . --force --ai claude

# 或者
specify init --here --force --ai claude
```

CLI 会检查你是否安装了 Claude Code、Gemini CLI、Cursor CLI、Qwen CLI、opencode、Codex CLI 或 Amazon Q Developer CLI。若未安装，或你希望在不检查工具的情况下获取模板，请在命令中加入 `--ignore-agent-tools`：

```bash
specify init <project_name> --ai claude --ignore-agent-tools
```

### 步骤 1：确立项目原则

进入项目文件夹并运行你的 AI 代理。以下示例使用 `claude`。

![为 Claude Code 启动开发环境](./media/bootstrap-claude-code.gif)

当你看到 `/speckit.constitution`、`/speckit.specify`、`/speckit.plan`、`/speckit.tasks` 与 `/speckit.implement` 命令可用时，说明配置正确。

第一步是使用 `/speckit.constitution` 命令确立项目的治理原则。这能确保后续各阶段决策的一致性：

```text
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements. Include governance for how these principles should guide technical decisions and implementation choices.
```

该步骤将在 `.specify/memory/constitution.md` 中创建或更新项目的基础指导文档，供 AI 代理在规格、计划与实现阶段参考。

### 步骤 2：创建项目规格

有了项目原则后，你可以创建功能规格。使用 `/speckit.specify` 并提供你要开发的项目的具体需求。

>[!IMPORTANT]
>尽可能明确你要构建的“内容与目的”。**此时不要关注技术栈。**

示例提示：

```text
Develop Taskify, a team productivity platform. It should allow users to create projects, add team members,
assign tasks, comment and move tasks between boards in Kanban style. In this initial phase for this feature,
let's call it "Create Taskify," let's have multiple users but the users will be declared ahead of time, predefined.
I want five users in two different categories, one product manager and four engineers. Let's create three
different sample projects. Let's have the standard Kanban columns for the status of each task, such as "To Do,"
"In Progress," "In Review," and "Done." There will be no login for this application as this is just the very
first testing thing to ensure that our basic features are set up. For each task in the UI for a task card,
you should be able to change the current status of the task between the different columns in the Kanban work board.
You should be able to leave an unlimited number of comments for a particular card. You should be able to, from that task
card, assign one of the valid users. When you first launch Taskify, it's going to give you a list of the five users to pick
from. There will be no password required. When you click on a user, you go into the main view, which displays the list of
projects. When you click on a project, you open the Kanban board for that project. You're going to see the columns.
You'll be able to drag and drop cards back and forth between different columns. You will see any cards that are
assigned to you, the currently logged in user, in a different color from all the other ones, so you can quickly
see yours. You can edit any comments that you make, but you can't edit comments that other people made. You can
delete any comments that you made, but you can't delete comments anybody else made.
```

输入该提示后，你应能看到 Claude Code 开始起草计划与规格，并触发内置脚本来初始化仓库。

完成后，会创建一个新分支（例如 `001-create-taskify`），并在 `specs/001-create-taskify` 目录下生成新的规格文档。

该规格应包含一组用户故事与功能需求（来源于模板）。

此时你的项目目录大致如下：

```text
└── .specify
    ├── memory
    │  └── constitution.md
    ├── scripts
    │  ├── check-prerequisites.sh
    │  ├── common.sh
    │  ├── create-new-feature.sh
    │  ├── setup-plan.sh
    │  └── update-claude-md.sh
    ├── specs
    │  └── 001-create-taskify
    │      └── spec.md
    └── templates
        ├── plan-template.md
        ├── spec-template.md
        └── tasks-template.md
```

### 步骤 3：澄清功能规格（规划前必做）

在有了初版规格后，你应澄清首次生成中未被准确捕捉的需求点。

在生成技术计划之前，先运行结构化澄清工作流，能有效减少后续返工。

推荐顺序：

1. 使用结构化的 `/speckit.clarify` —— 序列化、基于覆盖率的提问方式，并将答案记录在“Clarifications”章节。
2. 若仍存在模糊之处，可再进行一次自由的补充澄清。

如果你有意跳过澄清步骤（例如 Spike 或探索性原型），请明确说明，以免代理因缺少澄清而中断。

在运行 `/speckit.clarify` 后，如仍需自由补充，可参考如下示例：

```text
For each sample project or project that you create there should be a variable number of tasks between 5 and 15
tasks for each one randomly distributed into different states of completion. Make sure that there's at least
one task in each stage of completion.
```

你还可以让 Claude Code 校验 **Review & Acceptance Checklist**，将满足条件的条目勾选，未满足的保留未选中。可使用如下提示：

```text
Read the review and acceptance checklist, and check off each item in the checklist if the feature spec meets the criteria. Leave it empty if it does not.
```

与 Claude Code 的互动是澄清与提问规格的绝佳机会——**不要把它的首次回答当成最终结果**。

### 步骤 4：生成计划

现在你可以明确技术栈与其他技术要求了。你可以使用 `/speckit.plan`（模板已内置）并提供如下提示：

```text
We are going to generate this using .NET Aspire, using Postgres as the database. The frontend should use
Blazor server with drag-and-drop task boards, real-time updates. There should be a REST API created with a projects API,
tasks API, and a notifications API.
```

该步骤会输出多份实现细节文档，目录结构大致如下：

```text
.
├── CLAUDE.md
├── memory
│  └── constitution.md
├── scripts
│  ├── check-prerequisites.sh
│  ├── common.sh
│  ├── create-new-feature.sh
│  ├── setup-plan.sh
│  └── update-claude-md.sh
├── specs
│  └── 001-create-taskify
│      ├── contracts
│      │  ├── api-spec.json
│      │  └── signalr-spec.md
│      ├── data-model.md
│      ├── plan.md
│      ├── quickstart.md
│      ├── research.md
│      └── spec.md
└── templates
    ├── CLAUDE-template.md
    ├── plan-template.md
    ├── spec-template.md
    └── tasks-template.md
```

请检查 `research.md` 以确保技术栈符合你的指示。若有不妥之处，可让 Claude Code 进行修正；甚至可以让它检查本地是否安装了相应平台/框架（如 .NET）。

此外，若所选技术栈变化较快（如 .NET Aspire、部分 JS 框架），你可以让 Claude Code 进一步调研：

```text
I want you to go through the implementation plan and implementation details, looking for areas that could
benefit from additional research as .NET Aspire is a rapidly changing library. For those areas that you identify that
require further research, I want you to update the research document with additional details about the specific
versions that we are going to be using in this Taskify application and spawn parallel research tasks to clarify
any details using research from the web.
```

在此过程中，若你发现 Claude Code 的调研方向跑偏了，可以用如下提示“纠偏”：

```text
I think we need to break this down into a series of steps. First, identify a list of tasks
that you would need to do during implementation that you're not sure of or would benefit
from further research. Write down a list of those tasks. And then for each one of these tasks,
I want you to spin up a separate research task so that the net results is we are researching
all of those very specific tasks in parallel. What I saw you doing was it looks like you were
researching .NET Aspire in general and I don't think that's gonna do much for us in this case.
That's way too untargeted research. The research needs to help you solve a specific targeted question.
```

>[!NOTE]
>Claude Code 可能会“过度热心”，加入你并未要求的组件。请让它解释理由与来源。

### 步骤 5：由 Claude Code 校验计划

计划完成后，让 Claude Code 通读并检查是否存在缺漏：

```text
Now I want you to go and audit the implementation plan and the implementation detail files.
Read through it with an eye on determining whether or not there is a sequence of tasks that you need
to be doing that are obvious from reading this. Because I don't know if there's enough here. For example,
when I look at the core implementation, it would be useful to reference the appropriate places in the implementation
details where it can find the information as it walks through each step in the core implementation or in the refinement.
```

这能帮助你完善实现计划，避免 Claude Code 在规划阶段的疏漏。完成初版修订后，再让它按清单复核一次，随后进入实现阶段。

如果你安装了 [GitHub CLI](https://docs.github.com/en/github-cli/github-cli)，也可以让 Claude Code 直接从当前分支向 `main` 创建一个带有详细描述的 PR，确保工作可追踪。

>[!NOTE]
>在让代理实现之前，也值得让 Claude Code 检查是否存在“过度设计”。若有，请让它进行合理化简。确保它遵循[宪章/原则](base/memory/constitution.md)作为制定计划的基石。

### 步骤 6：使用 /speckit.tasks 生成任务分解

在计划得到验证后，可将其分解为可执行的任务序列。使用 `/speckit.tasks` 自动从实现计划生成详细任务分解：

```text
/speckit.tasks
```

该步骤会在你的功能规格目录中创建 `tasks.md`，其中包含：

- 按用户故事组织的任务分解 —— 每个用户故事对应独立的实现阶段及任务集
- 依赖管理 —— 按依赖顺序排列任务（例如先模型、再服务、再端点）
- 并行执行标记 —— 可并行的任务标记为 `[P]`，以优化开发流程
- 文件路径规范 —— 每个任务包含明确的实现文件路径
- 测试优先（TDD）结构 —— 如要求编写测试，则在实现前生成测试任务
- 检查点校验 —— 每个用户故事阶段包含独立功能的校验检查点

生成的 `tasks.md` 为 `/speckit.implement` 命令提供清晰的施工蓝图，确保系统化实现并维持代码质量，同时允许按用户故事逐步交付。

### 步骤 7：实施

准备就绪后，使用 `/speckit.implement` 执行实现计划：

```text
/speckit.implement
```

`/speckit.implement` 将会：

- 校验前置条件（宪章/原则、规格、计划与任务）
- 解析 `tasks.md` 中的任务分解
- 按依赖顺序与并行标记执行任务
- 遵循任务计划中定义的 TDD 方式
- 提供进度更新并妥善处理错误

>[!IMPORTANT]
>AI 代理将会执行本地 CLI 命令（如 `dotnet`、`npm` 等）——请确保你的机器已安装所需工具。

实现完成后，运行应用并解决任何可能未在 CLI 日志中体现的运行时错误（例如浏览器控制台错误）。你可以将这些错误复制回 AI 代理以协助定位与修复。

</details>

---

<a id="-troubleshooting"></a>

## 🔍 故障排查

### Linux 下的 Git Credential Manager

如果你在 Linux 上遇到 Git 认证问题，可以安装 Git Credential Manager：

```bash
#!/usr/bin/env bash
set -e
echo "Downloading Git Credential Manager v2.6.1..."
wget https://github.com/git-ecosystem/git-credential-manager/releases/download/v2.6.1/gcm-linux_amd64.2.6.1.deb
echo "Installing Git Credential Manager..."
sudo dpkg -i gcm-linux_amd64.2.6.1.deb
echo "Configuring Git to use GCM..."
git config --global credential.helper manager
echo "Cleaning up..."
rm gcm-linux_amd64.2.6.1.deb
```

<a id="-maintainers"></a>

## 👥 维护者

- Den Delimarsky（[@localden](https://github.com/localden)）
- John Lam（[@jflam](https://github.com/jflam)）

<a id="-support"></a>

## 💬 支持

如需支持，请创建一个[GitHub Issue](https://github.com/github/spec-kit/issues/new)。欢迎你提交错误报告、功能请求，以及关于规格驱动开发使用方法的问题。

<a id="-acknowledgements"></a>

## 🙏 致谢

本项目深受并基于 [John Lam](https://github.com/jflam) 的工作与研究启发。

<a id="-license"></a>

## 📄 许可证

本项目基于 MIT 开源许可证发布。完整条款请参阅 [LICENSE](./LICENSE) 文件。
