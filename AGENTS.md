# Agent Instructions

This repo hosts kaup release artifacts via GitHub Releases. See [`README.md`](./README.md) for details.

## Agent skills

### Issue tracker

Issues and specs live as GitHub issues. Use `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical roles mapped to `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout. Read `CONTEXT.md` at the repo root if it exists; ADRs live in `kaup-design/docs/superpowers/adr/`. See `docs/agents/domain.md`.

## 仓家族（五仓）

kaup 是**五仓项目**，五仓 clone 到同一父目录（如 `~/workspace/kaup/`），职责互不重叠：

| 仓 | 角色 | 技术栈 |
|----|------|--------|
| `kaup-design` | 设计 SSOT：spec / plan / ADR / runbook | Markdown |
| `kaup-core` | 后端 Python monorepo（api-proxy / egress-proxy / kaup-platform / kaup-platform-internal / kaup-leads / kaup-profiling / kaup-enrich / kaup-cli / kaupctl / kaup-common / kaup-hermes-gate） | Python 3.13 · uv · FastAPI · LangGraph · alembic |
| `kaup-frontend` | 桌面端 Thin Client + 设计系统 | Tauri v2 · Vue 3 + Vite · pnpm + Turborepo |
| `kaup-artifacts`（本仓） | 发版产物分发（desktop installer / CLI installers（+ standalone ZIP）） | GitHub Releases |
| `kaup-ai-website` | 官网（营销站 / 内容站 / 下载与安装入口） | Node 26 · Nuxt 4 · Nuxt UI · Nuxt Content · Tailwind 4 · pnpm |

**成员判据**：家族成员 = kaup 主产品线、其设计决策由 `kaup-design` 承载、其代码或产物由该仓自身承载。
家族外仓（同 org 的原型 / 独立产品 / 工具仓）不列入本表。

**口径（定死）**：**五仓** = design / core / frontend / artifacts / ai-website 五个 git 仓；**日常开发提交通常只落四仓**
（design / core / frontend / ai-website）——**本仓只托管 GitHub Releases 产物，几乎不产生提交**。
历史 journal / commit message 里的「三仓」指五仓化之前的旧口径（design / core / frontend），**不代表本仓不存在**。

**跨仓纪律**：

- 所有 git / ls / grep 命令**显式带绝对路径**（`git -C /path/to/kaup-<repo> …`），防 cwd 漂移。
- **代码改动在 `kaup-core` / `kaup-frontend` / `kaup-ai-website` 执行**；设计与决策落 `kaup-design`（设计 SSOT）——本仓不复制其正文，
  只留指针（见该仓 `AGENTS.md` §3）。
- 本表 SSOT = `kaup-design/AGENTS.md` §1。改这里必须同步改那边（**重复处有锚，漂移可检**）。

> **工作区边界**：`AGENTS.md` 的发现范围是「仓根 → 当前工作目录」，**结构上到不了仓外的兄弟仓**；
> 兄弟仓的 `AGENTS.md` 需显式读取。要把兄弟仓并入工作区：Codex 传 `--add-dir <DIR>`，
> Claude Code 配 `permissions.additionalDirectories`。
