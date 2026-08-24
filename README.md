# pi-config

开箱即用的 [Pi](https://pi.dev) 终端编码代理配置 —— **21 个插件 · 16 个 Skill · 1 个 MCP Server · 双层记忆系统**，一条命令完整复刻。

[![License: MIT](https://img.shields.io/badge/license-MIT-success)](LICENSE)
[![Pi](https://img.shields.io/badge/Pi-0.84.2-8A2BE2)](https://pi.dev)
[![Plugins](https://img.shields.io/badge/plugins-21-blue)](#-插件目录21-个)
[![Skills](https://img.shields.io/badge/skills-16-green)](#-skill-清单16-个)
[![MCP](https://img.shields.io/badge/MCP-1-orange)](#-mcp-integration)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#-compatibility)

仓库里的每一样东西都经过实际使用筛选，目标是把 Pi 打造成一个**能多代理协作、能省 token、能跑浏览器、有长期记忆**的全能终端编码代理。

**导航**：[Compatibility](#-compatibility) · [Quick Start](#-quick-start) · [插件目录](#-插件目录21-个) · [Skill 清单](#-skill-清单16-个) · [MCP](#-mcp-integration) · [记忆系统](#-双层记忆系统) · [配置文件](#️-配置文件速查) · [Limitations](#️-limitations)

---

## 😡 The Problem

裸装一个 AI 编码代理，第一天很兴奋，第三天就露馅：

> **You**：帮我重构这个模块，顺便按我上次的偏好来
> **Pi**：我没有浏览器工具查不了文档，也记不住你上次的偏好——每次新会话我都从零开始。
> **You**：……那我要你这"代理"干嘛？

痛点拆开就是四件事：**没有工具生态、token 烧得快、没有跨会话记忆、Windows 下体验割裂**。这个仓库一次性解决。

---

## ✅ The Solution

一条命令装好整套生态：

```bash
bash install.sh
```

```
📦 安装 21 个 Pi 插件包...
  → pi install npm:pi-subagents        ✓
  → pi install npm:context-mode        ✓
  ...共 21 个
🔌 配置 MCP servers...
  ✓ 已合并 MCP server: context7       （已有配置不覆盖）
✅ 安装完成！
```

装完即得：子代理编排 + AST 级代码理解 + 大输出沙箱省 token + Playwright 浏览器 + FTS5 双层记忆。

---

## 📊 Compatibility

| 环境 | 状态 | 说明 |
| --- | --- | --- |
| Windows 11 + PowerShell 7 | ✅ 作者日常环境 | 含 pwsh-adapter / path-guard 适配 |
| macOS / Linux | ✅ 支持 | `install.sh` 原生支持 |
| Git Bash / MSYS2 | ✅ 支持 | Windows 备选方案 |
| 模型 Provider / API Key | ⚠️ 需自配 | 用 `pi config` 配置，因人而异不入库 |
| Pi 版本 | 0.84.2 | 其他版本未测试 |

---

## 🚀 Quick Start

### 30 秒版（已装 Pi）

```bash
git clone https://github.com/Lumjiel/pi-config.git && cd pi-config && bash install.sh
# 重启 pi 即生效
```

### 分步版

```bash
# 0. 还没装 Pi？先装本体
npm install -g @earendil-works/pi-coding-agent

# 1. 克隆本仓库
git clone https://github.com/Lumjiel/pi-config.git
cd pi-config

# 2. 一键安装：21 个插件 + 合并 MCP 配置
bash install.sh

# 3. 配置你自己的模型 provider 和 key（脚本不做这步）
pi config

# 4. 重启 pi，让所有插件和 MCP server 生效
```

---

## 🧩 插件目录（21 个）

### 🤖 代理编排与工作流

| 插件 | 作用 | 仓库 |
| --- | --- | --- |
| `pi-subagents` | 子代理委派框架，single / chain / parallel / async / forked 五种模式 | [nicobailon/pi-subagents](https://github.com/nicobailon/pi-subagents) |
| `@narumitw/pi-goal` | `/goal` 目标驱动，自主推进长任务，遇阻塞主动停下 | [narumiruna/pi-extensions](https://github.com/narumiruna/pi-extensions) |
| `@narumitw/pi-plan-mode` | 只读计划模式，先出方案讨论清楚再动手 | 同上 |
| `@narumitw/pi-github-pr` | 终端里看 PR review / checks / comment | 同上 |
| `@juicesharp/rpiv-todo` | 实时浮层 todo list，扛得住 `/reload` 和上下文压缩 | [juicesharp/rpiv-mono](https://github.com/juicesharp/rpiv-mono) |

### 🔍 代码智能与上下文管理

| 插件 | 作用 | 仓库 |
| --- | --- | --- |
| `pi-lens` | AST 级代码理解：ast-grep 结构化搜索替换、tree-sitter 检查、LSP 诊断 | [apmantza/pi-lens](https://github.com/apmantza/pi-lens) |
| `context-mode` | 省 token 核心：大输出路由进沙箱处理只回摘要，内置 FTS5 知识库 | [mksglu/context-mode](https://github.com/mksglu/context-mode) |

> context-mode 效果实测：分析 47 个源文件，直接 read 要烧 ~700KB，走它只回 ~3.6KB。

### 🌐 浏览、检索与外部接入

| 插件 | 作用 | 仓库 |
| --- | --- | --- |
| `pi-web-access` | Web 全家桶：6 引擎搜索、URL 转 markdown、YouTube 转录、PDF 提取 | [nicobailon/pi-web-access](https://github.com/nicobailon/pi-web-access) |
| `pi-playwright` | 浏览器自动化：开页面、填表单、点击、截图、看 console/network | [guwidoe/pi-playwright](https://github.com/guwidoe/pi-playwright) |
| `pi-mcp-adapter` | 接入任意 MCP server，支持 OAuth、安全审查、懒启动 | [nicobailon/pi-mcp-adapter](https://github.com/nicobailon/pi-mcp-adapter) |
| `pi-marketplace` | 在 Pi 里直接搜索、审计、安装 npm 上的 pi 包 | [pi.dev/packages](https://pi.dev/packages/pi-marketplace) |

### 🪟 Windows / PowerShell 适配

| 插件 | 作用 | 仓库 |
| --- | --- | --- |
| `@99percentpeople/pi-pwsh-adapter` | 让 Pi 用原生 PowerShell 7 跑命令而非 cmd.exe，Windows 必装 | [99percentpeople/pi-pwsh-adapter](https://github.com/99percentpeople/pi-pwsh-adapter) |
| `pi-path-guard` | 防 Pi 意外写入系统目录或越界操作 | [nicobailon/pi-path-guard](https://github.com/nicobailon/pi-path-guard) |

### ✨ 实用工具与主题

| 插件 | 作用 | 仓库 |
| --- | --- | --- |
| `pi-simplify` | 审最近改动代码的清晰度 / 一致性 / 可维护性 | [MattDevy/pi-extensions](https://github.com/MattDevy/pi-extensions) |
| `@firstpick/pi-prompts-git-pr` | prompt 模板集：提交信息、PR 描述、review 流程，`/` 直接唤起 | [Firstp1ck/pi-coding-agent-forge](https://github.com/Firstp1ck/pi-coding-agent-forge) |
| `@firstpick/pi-skill-deep-research` | 两阶段深度研究流程，带 schema / policy 校验 | 同上 |
| `pi-trash` | 安全删除：`rm` 改为移入回收站，防误删不可恢复 | [nicobailon/pi-trash](https://github.com/nicobailon/pi-trash) |
| `pi-catppuccin-tui` | Catppuccin 暖色调主题集合 | [nicobailon/pi-catppuccin-tui](https://github.com/nicobailon/pi-catppuccin-tui) |
| `pi-hashline-edit-pro` | 行级哈希锚点编辑增强，大文件精准替换 | [nicobailon/pi-hashline-edit-pro](https://github.com/nicobailon/pi-hashline-edit-pro) |

---

## 🦞 Skill 清单（16 个）

全部来自已安装的 npm 包，装好插件自动获得：

| 类别 | Skill | 来源包 |
| --- | --- | --- |
| 代理编排 | `pi-subagents` | pi-subagents |
| 研究/浏览 | `librarian`、`deep-research` | pi-web-access / @firstpick |
| 浏览器 | `playwright-browser` | pi-playwright |
| 上下文/知识库 | `context-mode`、`ctx-search`、`ctx-index`、`ctx-stats`、`ctx-purge`、`ctx-insight`、`ctx-doctor`、`ctx-upgrade` | context-mode |
| 代码智能 | `pi-lens-ast-grep`、`pi-lens-lsp-navigation`、`pi-lens-write-ast-grep-rule`、`pi-lens-write-tree-sitter-rule` | pi-lens |

---

## 🤖 MCP Integration

唯一的 MCP server 是 Upstash 的 [Context7](https://github.com/upstash/context7)：给模型实时拉取第三方库**最新文档**，避免用过时的训练知识写代码。

配置在 [`mcp.json`](mcp.json)，安装脚本自动合并进 `~/.config/mcp/mcp.json`（不覆盖已有 server）：

```json
{
  "command": "npx",
  "args": ["-y", "@upstash/context7-mcp@latest"],
  "lifecycle": "lazy"
}
```

`"lifecycle": "lazy"` —— 按需启动，不常驻，省资源。

---

## 🧠 双层记忆系统

| 层 | 实现 | 定位 | 存储 |
| --- | --- | --- | --- |
| L2 精华记忆 | pi-hermes-memory | 跨会话持久化的决策/偏好/教训，FTS5 全文检索 | `~/.pi/agent/pi-hermes-memory/` |
| L3 知识库 | context-mode | 大文档/手册索引，原文不进上下文只回匹配窗口 | `ctx_index` / `ctx_search` 管理 |

L2 共 **30 个 .md 文件**，分四类 target：`memory`（全局）、`user`（用户画像）、`project`（项目级）、`failure`（失败教训）。

关键文件：

| 文件 | 内容 |
| --- | --- |
| `MEMORY.md` | 全局精华：工具链、环境、关键决策 |
| `USER.md` | 用户画像：偏好、沟通风格、工作习惯 |
| `failures.md` | 失败教训：踩过的坑和修复方法 |
| `lessons.md` | 跨项目可复用的操作技巧 |
| `windows-*.md` | Windows 运维系列（防火墙/更新/Bat 编码等） |

配置项：`autoConsolidate: false`（手动固化）、`correctionDetection: true`(自动识别纠错)。

---

## ⚙️ 配置文件速查

| 文件 | 作用 |
| --- | --- |
| `~/.pi/agent/settings.json` | 全局设置：packages 列表、主题、默认模型 |
| `~/.pi/agent/models.json` | provider 和模型定义 |
| `~/.pi/agent/skills/` | 全局自定义 skill |
| `~/.config/mcp/mcp.json` | MCP server 配置 |
| `.pi/settings.json` | 项目级设置（可跟团队共享） |

自定义暗色主题 `vivid-night` 存放于 `themes/vivid-night.json`，安装后自动加载。

---

## 📁 Structure

```
pi-config/
├── install.sh      # 一键安装：装 21 个插件 + 合并 MCP 配置
├── config.json     # 机器可读完整配置（plugins/skills/MCP/UI/memory）
├── mcp.json        # MCP server 定义（context7）
└── README.md       # 本文件
```

---

## ⚠️ Limitations

| 限制 | Workaround |
| --- | --- |
| 不含模型 provider / API key | `pi config` 自行配置 |
| Windows 适配插件在 mac/Linux 上多余 | 不想要就删掉 settings.json 里对应行 |
| 插件版本随上游更新，行为可能变化 | `pi install npm:<pkg>@<version>` pin 版本 |
| 非 0.84.2 的 Pi 版本未测试 | 遇问题先对齐 Pi 版本再排查 |

---

## 🙏 Acknowledgements

本仓库是插件的组装与调优，核心能力全部来自以上开源作者，感谢他们的工作。想自己动手扩展 Pi？三档改造路径：写 **Skill**（零代码沉淀流程知识）→ 写 **MCP Server**（接入外部服务）→ 写 **Extension**（订阅事件、注册工具的最强改造）。

## 📜 License

[MIT](LICENSE)
