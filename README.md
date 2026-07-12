# kaup-artifacts

kaup **public 发布通道** —— desktop 安装包 + auto-updater manifest 的公开发布仓。

## 为什么需要这个仓

kaup 源码仓（kaup-core / kaup-frontend / kaup-design）都是 **private** 保护知识产权。但 Tauri auto-updater 客户端要**匿名访问** updater manifest（latest.json）+ 更新包（.dmg/.msi/.AppImage），private 仓做不到。kaup-artifacts 是 public，供客户端零认证拉取。

## 安全模型（不靠 repo 访问控制，靠 Tauri 签名验签）

- `TAURI_SIGNING_PUBLIC_KEY` 内置 desktop 客户端（tauri.conf.json）
- 每个更新包 `.sig` 签名（CI 用 `TAURI_SIGNING_PRIVATE_KEY` 签）
- 客户端拉更新 → 验签 → 通过才替换
- 即使有人 fork/复制本仓产物，没有私钥签不出客户端接受的更新
- 同 kaup "KAUP_API_KEY 应用层认证" 哲学（不依赖网络层）

## 仓内容

- **git 本身**：只放 README + .gitignore（无源码、无敏感信息）
- **GitHub Releases**（W5-4 起）：每个 release tag（v0.1.0）一个 Release，assets 含：
  - macOS: `kaup_<ver>_universal.dmg` + `.sig`
  - Ubuntu: `kaup_<ver>_amd64.AppImage` + `.deb` + `.sig`
  - Windows: `kaup_<ver>_x64.msi` + `.nsis` + `.sig`
  - `latest.json`（updater manifest：各平台当前版本 + 签名 + 下载 URL）

## 客户端更新流程

```
desktop 启动 → 查 https://github.com/kaup-ai/kaup-artifacts/releases/latest/download/latest.json
            → 对比当前版本 → 有新版则下载 + 验签（TAURI_SIGNING_PUBLIC_KEY）→ 替换 + 重启
```

## 决策依据

- ADR-0026（D2 三仓体系 + D7 内置 updater + D9 kaup-artifacts 跨仓发布）
- 首次安装下载链接：kaup-design 仓 ONBOARDING.md / 下载页指向本仓 Releases

## 仓维护

- 不要在本仓 commit 二进制（.dmg/.msi/.AppImage）——它们进 Releases attachments，不进 git
- Releases 由 kaup-frontend CI 跨仓 push（KAUPT_ARTIFACTS_TOKEN = GitHub App token，W5-4 配）
- 历史 Release 保留（updater 回滚 + 客户端版本溯源）
