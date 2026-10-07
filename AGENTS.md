# Agent Instructions

This repo hosts kaup release artifacts via GitHub Releases. See [`README.md`](./README.md) for details.

## Agent skills

### Issue tracker

Issues and specs live as GitHub issues. Use `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical roles mapped to `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout. Read `CONTEXT.md` at the repo root if it exists; ADRs live in `kaup-design/docs/superpowers/adr/`. See `docs/agents/domain.md`.

## 仓家族（四仓）

kaup 是**四仓项目**，四仓 clone 到同一父目录（如 `~/workspace/kaup/`），职责互不重叠：

| 仓 | 角色 | 技术栈 |
|----|------|--------|
| `kaup-design` | 设计 SSOT：spec / plan / ADR / runbook | Markdown |
| `kaup-core` | 后端 Python monorepo（api-proxy / kaup-platform / kaup-platform-internal / kaup-leads / kaup-profiling / kaup-enrich / kaup-cli / kaup-common / kaup-hermes-gate） | Python 3.13 · uv · FastAPI · LangGraph · alembic |
| `kaup-frontend` | 桌面端 Thin Client + 设计系统 | Tauri v2 · Vue 3 + Vite · pnpm + Turborepo |
| `kaup-artifacts`（本仓） | 发版产物分发（desktop installer / CLI binary） | GitHub Releases |

**口径（定死）**：**四仓** = design / core / frontend / artifacts 四个 git 仓；**日常开发提交通常只落三仓**
（design / core / frontend）——**本仓只托管 GitHub Releases 产物，几乎不产生提交**。
历史 journal / commit message 里的「三仓」指后者，**不代表本仓不存在**。

**跨仓纪律**：

- 所有 git / ls / grep 命令**显式带绝对路径**（`git -C /path/to/kaup-<repo> …`），防 cwd 漂移。
- **代码改动在 `kaup-core` / `kaup-frontend` 执行**；设计与决策落 `kaup-design`（设计 SSOT）——本仓不复制其正文，
  只留指针（见该仓 `AGENTS.md` §3）。
- 本表 SSOT = `kaup-design/AGENTS.md` §1。改这里必须同步改那边（**重复处有锚，漂移可检**）。

> **工作区边界**：`AGENTS.md` 的发现范围是「仓根 → 当前工作目录」，**结构上到不了仓外的兄弟仓**；
> 兄弟仓的 `AGENTS.md` 需显式读取。要把兄弟仓并入工作区：Codex 传 `--add-dir <DIR>`，
> Claude Code 配 `permissions.additionalDirectories`。
