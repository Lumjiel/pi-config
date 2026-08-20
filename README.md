# 我的 Pi Coding Agent 配置

[![License: MIT](https://img.shields.io/badge/license-MIT-success)](LICENSE)
[![Pi](https://img.shields.io/badge/Pi-0.84.2-8A2BE2)](https://pi.dev)
[![Plugins](https://img.shields.io/badge/plugins-21-blue)](#六插件目录21个按用途分组)
[![MCP](https://img.shields.io/badge/MCP-1-orange)](#八mcp-server1个)
[![Platform](https://img.shields.io/badge/platform-Windows%2011%20%7C%20PowerShell%20%7C%20Git%20Bash-lightgrey)](#五快速上手3步)
[![Memory](https://img.shields.io/badge/memory-30%20files-ff69b4)](#九记忆系统两层的)

一份可直接复刻的 [Pi](https://pi.dev) 配置，整理成教程形式分享。包含 **21 个插件、16 个全局 Skill、1 个 MCP server、双层记忆系统**，配一键安装脚本。
仓库里的每样东西都经过实际使用筛选，目标是把 Pi 打造成一个能多代理协作、能省 token、能跑浏览器、能操作邮件、有长期记忆的全能终端编码代理。

---

## 一、Pi 是什么？

[Pi](https://pi.dev)（npm 包 `@earendil-works/pi-coding-agent`）是一个开源的终端 AI 编码代理。你给它一个任务，它在你本地的项目里读写文件、跑命令、调用工具来完成任务。

如果你还没装 Pi，先：

```bash
npm install -g @earendil-works/pi-coding-agent
```

然后跑 `pi` 进入交互界面。本仓库的所有插件都假设你已经有一个能正常工作的 Pi。

---

## 二、Pi 的整体架构

理解架构有助于你挑选和编写插件。Pi 是一个**插件驱动的代理循环**，核心分层：

```
┌─────────────────────────────────────────────────────┐
│  TUI / RPC / Print   ← 用户交互层（终端界面 / API）  │
├─────────────────────────────────────────────────────┤
│  Agent Loop          ← 代理循环：收 prompt → 调模型  │
│                         → 执行工具 → 回传结果         │
├─────────────────────────────────────────────────────┤
│  Providers / Models  ← 多模型抽象：OpenAI / Anthropic │
│                         / 本地 / 自定义 provider      │
├─────────────────────────────────────────────────────┤
│  Tools               ← 模型可调用的工具：bash/read/   │
│                         edit/web_search/mcp/自定义    │
├─────────────────────────────────────────────────────┤
│  Extensions          ← TypeScript 模块，能订阅事件、  │
│                         注册工具/命令/快捷键/主题      │
├─────────────────────────────────────────────────────┤
│  Skills / Prompts    ← Markdown 形式的「过程知识」，  │
│                         按需注入到模型上下文           │
├─────────────────────────────────────────────────────┤
│  Themes              ← JSON 配色方案                  │
├─────────────────────────────────────────────────────┤
│  Packages            ← 上面这些资源的分发单位          │
│                         （npm / git / 本地路径）       │
└─────────────────────────────────────────────────────┘
```

**关键概念：**

- **Extension（扩展）**：TypeScript 模块，最强力的改造方式。能订阅生命周期事件、注册自定义工具、注册命令、改系统提示词、自定义渲染。
- **Skill（技能）**：一个 `SKILL.md` 文件，描述「某类任务怎么做」。模型按需加载，跨会话存活。
- **Theme（主题）**：JSON 配色方案。本仓库包含一个自定义主题 `vivid-night`。
- **Package（包）**：把上面这些打包，通过 npm / git / 本地路径分发。`pi install` 就是装包。

---

## 三、Pi 的生态与包市场

### 包市场（pi.dev/packages）

Pi 有一个官方的 [包市场](https://pi.dev/packages)，所有在 `package.json` 里带 `pi-package` 关键字的 npm 包都会自动出现在上面。

```bash
pi install npm:@some/package
```

### 三种包来源

| 来源 | 格式 | 说明 |
| --- | --- | --- |
| npm | `npm:@scope/pkg@1.2.3` | 最常见，版本可 pin |
| git | `git:github.com/user/repo@v1` | 适合私有 / 未发布的包 |
| 本地路径 | `/abs/path` 或 `./rel/path` | 开发调试用 |

### 配置文件位置

| 文件 | 作用 |
| --- | --- |
| `~/.pi/agent/settings.json` | 全局设置：packages 列表、主题、默认模型等 |
| `~/.pi/agent/models.json` | provider 和模型定义 |
| `~/.pi/agent/extensions/*.ts` | 全局自定义扩展 |
| `~/.pi/agent/skills/` | 全局自定义 skill |
| `~/.config/mcp/mcp.json` | MCP server 配置 |
| `.pi/settings.json` | 项目级设置（可跟团队共享） |

---

## 四、如何改造 / 扩展 Pi

Pi 提供了从轻到重三档改造方式：

### 轻量：写一个 Skill

最简单的扩展。在 `~/.pi/agent/skills/my-skill/SKILL.md` 写一个 markdown，描述某类任务的标准流程。模型会按需加载它。
适合：沉淀「怎么发邮件」「怎么做代码审查」「怎么操作 HuggingFace」这类流程知识，零代码。

### 中量：装 / 写 MCP Server

MCP（Model Context Protocol）是跨 agent 的工具协议。写一个 MCP server，在 `~/.config/mcp/mcp.json` 注册，Pi 通过 `pi-mcp-adapter` 把它的工具接进来。

适合：接入外部服务（数据库、API、浏览器、文档源），工具跨 agent 复用。

### 重量：写一个 Extension

最强力的改造。创建 `~/.pi/agent/extensions/my-ext.ts`，订阅事件、注册工具、拦截调用。

适合：权限门禁、git checkpoint、自定义 compaction、外部集成、有状态工具。

---

## 五、快速上手（3 步）

```bash
# 1. 克隆本仓库
git clone https://github.com/Lumjiel/pi-config.git
cd pi-config

# 2. 一键安装（装 21 个插件 + 合并 MCP 配置）
bash install.sh

# 3. 重启 pi，让所有插件和 MCP server 生效
```

脚本会自动把 21 个插件用 `pi install` 装好，并把 MCP server 的配置合并进 `~/.config/mcp/mcp.json`。
> 安装完之后，模型的 provider / key 还需要你自己用 `pi config` 配一下——这部分因人而异，不在本仓库范围内。

---

## 六、插件目录（21 个，按用途分组）

### 🤖 代理编排与工作流

#### `pi-subagents`

子代理委派框架。让主代理可以把任务派给子代理，支持五种工作模式：single / chain / parallel / async / forked-context。适合做「先让一个代理研究、再让另一个代理实现、最后让第三个代理 review」这种复杂流程。

- 📦 仓库：<https://github.com/nicobailon/pi-subagents>

#### `@narumitw/pi-goal`

目标驱动模式。用 `/goal` 设一个目标，Pi 会自主推进直到完成，中途遇到阻塞会主动停下来等你。适合长任务。

- 📦 仓库：<https://github.com/narumiruna/pi-extensions>

#### `@narumitw/pi-plan-mode`

只读的计划模式。让模型先出方案、跟你讨论清楚，再进入执行。避免一上来就乱改文件。

- 📦 仓库：<https://github.com/narumiruna/pi-extensions>

### 🔍 代码智能与上下文管理

#### `pi-lens`

Pi 的「IDE 眼睛」。AST 级别的代码理解能力：ast-grep 结构化搜索/替换、tree-sitter 语法规则检查、LSP 诊断、符号搜索、模块报告。配套 4 个 skill。

- 📦 仓库：<https://github.com/apmantza/pi-lens>

#### `context-mode`

省 token 的核心插件。把大输出路由进沙箱，用代码处理，只把摘要返回给模型。内置 FTS5 全文检索知识库。效果：分析 47 个源文件，直接 read 要烧 ~700KB；走 context-mode 只回 ~3.6KB。配套 8 个 skill。

工具层级：
- **核心工具**：`ctx_execute` / `ctx_execute_file` — 大输出分析的首选
- **知识库**：`ctx_index` / `ctx_search` — 索引文档后按需检索
- **Web 索引**：`ctx_fetch_and_index` — 抓取 URL 并索引
- **批量执行**：`ctx_batch_execute` — 多命令并行
- **管理工具**：`ctx_stats` / `ctx_purge` / `ctx_insight` / `ctx_doctor` / `ctx_upgrade`

- 📦 仓库：<https://github.com/mksglu/context-mode>
#### `pi-hermes-memory`

跨会话记忆。让 Pi 记住你之前告诉过它的事（偏好、项目约定、踩过的坑），下次开新会话还能用上。记忆分 user / memory / project / failure 四类，可搜索。

- 📦 仓库：<https://github.com/chandra447/pi-hermes-memory>

### 🌐 浏览、检索与外部接入

#### `pi-web-access`

Web 访问全家桶：多引擎网页搜索（OpenAI / Brave / Exa / Tavily / Perplexity / Gemini）、URL 内容转 markdown、YouTube 转录、GitHub 仓库克隆、PDF 提取。配带 `librarian` skill。

- 📦 仓库：<https://github.com/nicobailon/pi-web-access>

#### `pi-playwright`

Playwright 浏览器自动化。让 Pi 能开浏览器、填表单、点按钮、截图、查 console/network。配带 `playwright-browser` skill。

- 📦 仓库：<https://github.com/guwidoe/pi-playwright>

#### `pi-mcp-adapter`

MCP 适配器。让 Pi 能连接任何 MCP server，支持 OAuth、安全审查、按需懒启动。

- 📦 仓库：<https://github.com/nicobailon/pi-mcp-adapter>

#### `pi-marketplace`

Pi 包市场入口。在 Pi 里直接搜索、查看详情、安全审计、安装 npm 上的 pi 包。

- 📦 仓库：<https://pi.dev/packages/pi-marketplace>

### 🖥️ Windows / PowerShell 适配

#### `@99percentpeople/pi-pwsh-adapter`

PowerShell 7 适配器。让 Pi 在 Windows 上用原生 PowerShell 跑命令，而不是默认的 cmd.exe。对 Windows 用户是必装。

- 📦 仓库：<https://github.com/99percentpeople/pi-pwsh-adapter>

#### `pi-path-guard`

路径守卫。防止 Pi 在 Windows 上意外写入系统目录或越界操作。

- 📦 仓库：<https://github.com/nicobailon/pi-path-guard>

### ✨ 实用工具与主题

#### `@juicesharp/rpiv-todo`

给模型的 todo list，渲染成实时浮层，扛得住 `/reload` 和会话压缩。多步骤任务进度可视化。

- 📦 仓库：<https://github.com/juicesharp/rpiv-mono>

#### `@narumitw/pi-github-pr`

在 Pi 里看 GitHub PR 的 review / checks / comment 状态，不用切浏览器。

- 📦 仓库：<https://github.com/narumiruna/pi-extensions>

#### `pi-simplify`

审最近改动的代码，从清晰度、一致性、可维护性角度给建议。改完代码跑一下，把烂味道扫干净。

- 📦 仓库：<https://github.com/MattDevy/pi-extensions>

#### `@firstpick/pi-prompts-git-pr`

一套可复用的 prompt 模板：提交信息、PR 描述、PR review 流程。直接 `/` 唤起对应模板。

- 📦 仓库：<https://github.com/Firstp1ck/pi-coding-agent-forge>

#### `@firstpick/pi-skill-deep-research`

带 `deep-research` skill：两阶段严谨研究流程，带 schema / policy 校验。适合需要多源证据的高 stakes 研究。

- 📦 仓库：<https://github.com/Firstp1ck/pi-coding-agent-forge>

#### `pi-trash`

安全删除。把 `rm` 替换成移到回收站，防止误删不可恢复的文件。

- 📦 仓库：<https://github.com/nicobailon/pi-trash>

#### `pi-catppuccin-tui`

Catppuccin 配色方案集合。一套和谐的暖色调主题，护眼看久了不累。

- 📦 仓库：<https://github.com/nicobailon/pi-catppuccin-tui>

#### `pi-hashline-edit-pro`

行级哈希锚点编辑增强。让 Pi 的 `replace` 工具在大文件里更精准地定位和替换代码块。

- 📦 仓库：<https://github.com/nicobailon/pi-hashline-edit-pro>

---

## 七、全局 Skill 清单（16 个）

所有 skill 都来自已安装的 npm 包，装好插件即自动获得。

| 类别 | Skill | 来源包 |
| --- | --- | --- |
| 代理编排 | `pi-subagents` | pi-subagents |
| 研究/浏览 | `librarian`、`deep-research` | pi-web-access / @firstpick |
| 浏览器 | `playwright-browser` | pi-playwright |
| 上下文/知识库 | `context-mode`、`ctx-search`、`ctx-index`、`ctx-stats`、`ctx-purge`、`ctx-insight`、`ctx-doctor`、`ctx-upgrade` | context-mode |
| 代码智能 | `pi-lens-ast-grep`、`pi-lens-lsp-navigation`、`pi-lens-write-ast-grep-rule`、`pi-lens-write-tree-sitter-rule` | pi-lens |
---

## 八、MCP Server（1 个）

配置在 `mcp.json`，安装脚本会合并到 `~/.config/mcp/mcp.json`。

### `context7`

Upstash 的 Context7 MCP。给模型实时拉取第三方库的**最新文档**，避免它用过时的训练知识写代码。

```json
{
  "command": "npx",
  "args": ["-y", "@upstash/context7-mcp@latest"],
  "lifecycle": "lazy"
}
```

设了 `"lifecycle": "lazy"`——按需启动，不常驻，省资源。

---

## 九、记忆系统（两层）

本仓库实现了**双层记忆架构**，让 Pi 既有快速检索的精华知识，又有海量的文档沉淀。

### L2：pi-hermes-memory（精华记忆）

跨会话持久化记忆，存储在 `~/.pi/agent/pi-hermes-memory/`。

- **30 个 .md 文件**，覆盖：环境配置、教训、偏好、项目约定、Windows 运维、网络路由等
- 分四类 target：`memory`（全局）、`user`（用户画像）、`project`（项目级）、`failure`（失败教训）
- 基于 SQLite FTS5 全文检索
- 配置：`autoConsolidate: false`、`correctionDetection: true`

关键文件：

| 文件 | 内容 |
| --- | --- |
| `MEMORY.md` | 全局精华：工具链、环境、关键决策 |
| `USER.md` | 用户画像：偏好、沟通风格、工作习惯 |
| `SYSTEM.md` | 系统环境：OS、Shell、已装工具 |
| `failures.md` | 失败教训：踩过的坑和修复方法 |
| `lessons.md` | 通用经验：跨项目可复用的操作技巧 |
| `environment-config.md` | 开发环境详细配置参考 |
| `python-environment.md` | Python 环境管理方案 |
| `web-routing.md` | 网络代理和路由规则 |
| `windows-*.md` | Windows 运维系列（防火墙/更新/Bat编码等） |

### L3：context-mode（知识库）

大文档和过程数据的索引层。

- 用 `ctx_index` 索引文档/网页/手册
- 用 `ctx_search` 按需检索
- 原始内容不进对话上下文，只回匹配的窗口
- 适合：大型 API 文档、框架源码分析、历史会话检索

---

## 十、自定义主题

### `vivid-night`

自定义暗色主题，存放在 `pi-agent/themes/vivid-night.json`。安装后 Pi 会自动加载。

---

## 十一、仓库内容

| 文件 | 说明 |
| --- | --- |
| `install.sh` | 一键安装脚本：装 21 个插件 + 合并 MCP 配置 |
| `config.json` | 机器可读的完整配置（plugins / skills / MCP / UI / tools / memory） |
| `mcp.json` | MCP server 配置 |
| `README.md` | 本文件 |
---

## 十二、装完之后怎么用

1. **先配模型**：跑 `pi config`，加你自己的 provider 和模型。本仓库不涉及这部分。
2. **重启 Pi**：让新装的插件和 MCP server 生效。
3. **试试 skill**：在 Pi 里直接描述任务，模型会自动匹配合适的 skill。
4. **双层记忆**：L2 自动生效；L3 需要时用 `ctx_index` 索引文档。

---

## License

MIT
