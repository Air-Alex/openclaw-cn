---
summary: "ClawdHub 指南：公共技能注册表 + CLI 工作流程"
read_when:
  - 向新用户介绍 ClawdHub 时
  - 安装、搜索或发布技能时
  - 解释 ClawdHub CLI 标志和同步行为时
---

# ClawdHub

ClawdHub 是 **OpenClaw 的公共技能注册表**。它是一项免费服务：所有技能都是公开的、开放的，并且对所有人可见，用于分享和重复使用。技能只是一个包含 `SKILL.md` 文件的文件夹（加上支持文本文件）。您可以在 Web 应用程序中浏览技能，或使用 CLI 来搜索、安装、更新和发布技能。

站点：[clawdhub.com](https://clawdhub.com)

## 适用人群（初学者友好）

如果您想为您的 OpenClaw 代理添加新功能，ClawdHub 是查找和安装技能的最简单方法。您无需了解后端的工作原理。您可以：

- 用自然语言搜索技能。
- 将技能安装到您的工作区。
- 稍后用一条命令更新技能。
- 通过发布来备份您自己的技能。

## 快速入门（非技术用户）

1) 安装 CLI（见下一节）。
2) 搜索您需要的内容：
   - `clawdhub search "日历"`
3) 安装技能：
   - `clawdhub install <skill-slug>`
4) 启动一个新的 OpenClaw 会话，使其获取新技能。

## 安装 CLI

选择其中一个：

```bash
npm i -g clawdhub
```

```bash
pnpm add -g clawdhub
```

## 如何融入 OpenClaw

默认情况下，CLI 将技能安装到您当前工作目录下的 `./skills` 中。如果配置了 OpenClaw 工作区，除非您覆盖 `--workdir`（或 `CLAWDHUB_WORKDIR`），否则 `clawdhub` 会回退到该工作区。OpenClaw 从 `<workspace>/skills` 加载工作区技能，并在**下一个**会话中获取它们。如果您已经使用 `~/.openclaw/skills` 或捆绑技能，工作区技能优先。

有关技能如何加载、共享和门控的更多详细信息，请参见
[技能](/tools/skills)。

## 服务提供的功能（特性）

- **公开浏览** 技能及其 `SKILL.md` 内容。
- **搜索** 由嵌入（向量搜索）驱动，不仅仅是关键词。
- **版本控制** 使用语义化版本、变更日志和标签（包括 `latest`）。
- **下载** 每个版本的 zip 文件。
- **星标和评论** 用于社区反馈。
- **审核** 钩子用于审批和审计。
- **CLI 友好 API** 用于自动化和脚本编写。

## CLI 命令和参数

全局选项（适用于所有命令）：

- `--workdir <dir>`：工作目录（默认：当前目录；回退到 OpenClaw 工作区）。
- `--dir <dir>`：技能目录，相对于工作目录（默认：`skills`）。
- `--site <url>`：站点基础 URL（浏览器登录）。
- `--registry <url>`：注册表 API 基础 URL。
- `--no-input`：禁用提示（非交互式）。
- `-V, --cli-version`：打印 CLI 版本。

认证：

- `clawdhub login`（浏览器流程）或 `clawdhub login --token <token>`
- `clawdhub logout`
- `clawdhub whoami`

选项：

- `--token <token>`：粘贴 API 令牌。
- `--label <label>`：存储浏览器登录令牌的标签（默认：`CLI token`）。
- `--no-browser`：不打开浏览器（需要 `--token`）。

搜索：

- `clawdhub search "query"`
- `--limit <n>`：最大结果数。

安装：

- `clawdhub install <slug>`
- `--version <version>`：安装特定版本。
- `--force`：如果文件夹已存在则覆盖。

更新：

- `clawdhub update <slug>`
- `clawdhub update --all`
- `--version <version>`：更新到特定版本（仅限单个 slug）。
- `--force`：当本地文件与任何已发布的版本不匹配时覆盖。

列表：

- `clawdhub list`（读取 `.clawdhub/lock.json`）

发布：

- `clawdhub publish <path>`
- `--slug <slug>`：技能 slug。
- `--name <name>`：显示名称。
- `--version <version>`：语义化版本。
- `--changelog <text>`：变更日志文本（可以为空）。
- `--tags <tags>`：逗号分隔的标签（默认：`latest`）。

删除/取消删除（仅限所有者/管理员）：

- `clawdhub delete <slug> --yes`
- `clawdhub undelete <slug> --yes`

同步（扫描本地技能 + 发布新/更新的技能）：

- `clawdhub sync`
- `--root <dir...>`：额外扫描根目录。
- `--all`：上传所有内容而不提示。
- `--dry-run`：显示将要上传的内容。
- `--bump <type>`：`patch|minor|major` 用于更新（默认：`patch`）。
- `--changelog <text>`：非交互式更新的变更日志。
- `--tags <tags>`：逗号分隔的标签（默认：`latest`）。
- `--concurrency <n>`：注册表检查（默认：4）。

## 代理的常见工作流程

### 搜索技能

```bash
clawdhub search "postgres 备份"
```

### 下载新技能

```bash
clawdhub install my-skill-pack
```

### 更新已安装的技能

```bash
clawdhub update --all
```

### 备份您的技能（发布或同步）

对于单个技能文件夹：

```bash
clawdhub publish ./my-skill --slug my-skill --name "我的技能" --version 1.0.0 --tags latest
```

一次扫描并备份多个技能：

```bash
clawdhub sync --all
```

## 高级细节（技术）

### 版本控制和标签

- 每次发布都会创建一个新的 **语义化版本** `SkillVersion`。
- 标签（如 `latest`）指向一个版本；移动标签可以让您回滚。
- 变更日志按版本附加，在同步或发布更新时可以为空。

### 本地更改与注册表版本

更新会使用内容哈希将本地技能内容与注册表版本进行比较。如果本地文件与任何已发布的版本不匹配，CLI 会在覆盖前询问（或在非交互式运行中需要 `--force`）。

### 同步扫描和回退根目录

`clawdhub sync` 首先扫描您当前的工作目录。如果没有找到技能，它会回退到已知的旧位置（例如 `~/openclawot/skills` 和 `~/.openclaw/skills`）。这是为了在没有额外标志的情况下找到较旧的技能安装。

### 存储和锁定文件

- 已安装的技能记录在工作目录下的 `.clawdhub/lock.json` 中。
- 认证令牌存储在 ClawdHub CLI 配置文件中（通过 `CLAWDHUB_CONFIG_PATH` 覆盖）。

### 遥测（安装计数）

当您登录时运行 `clawdhub sync`，CLI 会发送一个最小快照以计算安装次数。您可以完全禁用此功能：

```bash
export CLAWDHUB_DISABLE_TELEMETRY=1
```

## 环境变量

- `CLAWDHUB_SITE`：覆盖站点 URL。
- `CLAWDHUB_REGISTRY`：覆盖注册表 API URL。
- `CLAWDHUB_CONFIG_PATH`：覆盖 CLI 存储令牌/配置的位置。
- `CLAWDHUB_WORKDIR`：覆盖默认工作目录。
- `CLAWDHUB_DISABLE_TELEMETRY=1`：在 `sync` 上禁用遥测。
