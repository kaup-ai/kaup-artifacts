# kaup-artifacts

kaup **public 发布通道** —— desktop 安装包 + CLI 二进制 + auto-updater manifest 的公开发布仓。

## 为什么需要这个仓

kaup 源码仓（kaup-core / kaup-frontend / kaup-design）都是 **private** 保护知识产权。但 Tauri auto-updater 客户端要**匿名访问** updater manifest（latest.json）+ 更新包（.dmg/.msi/.AppImage），private 仓做不到。kaup-artifacts 是 public，供客户端零认证拉取。

## 安全模型（不靠 repo 访问控制，靠 Tauri 签名验签）

- `TAURI_SIGNING_PUBLIC_KEY` 内置 desktop 客户端（tauri.conf.json）
- 每个更新包 `.sig` 签名（CI 用 `TAURI_SIGNING_PRIVATE_KEY` 签）
- 客户端拉更新 → 验签 → 通过才替换
- 即使有人 fork/复制本仓产物，没有私钥签不出客户端接受的更新
- 同 kaup "KAUP_API_KEY 应用层认证" 哲学（不依赖网络层）

### CLI（kaup-cli-v<ver>）

- 不走 Tauri 签名（CLI 非 Tauri app）
- Phase-1：IT 信任 GitHub Release 源（kaup-ai/kaup-artifacts 官方）+ 可选 SHA256 checksum 核对
- Phase-2（FU-C1-5）：加 cosign/GPG 签名 + 强制 checksum，客户 verify 后部署
- 源码保护：Nuitka 编译为原生二进制（.pyc 反编译弱保护，但满足"不公开 .py 源码"）

## 仓内容

两条独立 release 线，各自版本号、各自 CI 发布：

### 1. desktop（业务人员，每人 1 台）
- release 命名：`kaup-desktop-v<ver>`（由 kaup-frontend CI 跨仓 push）
- assets：macOS `.dmg` + `.sig` / Ubuntu `.AppImage` + `.deb` + `.sig` / Windows `.msi` + `.nsis` + `.sig`
- `latest.json`（Tauri updater manifest：各平台当前版本 + 签名 + 下载 URL）

### 2. CLI（IT 共享服务器，1 台/公司）
- release 命名：`kaup-cli-v<ver>`（由 kaup-core CI 跨仓 push）
- assets：Nuitka 编译的单文件二进制 `kaup-<os>-<arch>`（Phase-1: `kaup-linux-amd64` 必须；mac/win 随后 FU-C1-3）
- 无 latest.json（服务器端工具，IT deliberate 升级，无 auto-updater D6）
- 源码仓 kaup-core **私有**；公开的只有 Nuitka 编译产物（二进制，非 .py 源码）→ 满足"不开源"

> git 本身只放 README + .gitignore（无源码、无敏感信息）。二进制进 Releases attachments，不进 git。

## 客户端流程

### desktop（业务人员）
desktop 启动 → 查 `https://github.com/kaup-ai/kaup-artifacts/releases/latest/download/latest.json` → 对比版本 → 有新版下载 + 验签（TAURI_SIGNING_PUBLIC_KEY）→ 替换重启。

### CLI（IT 共享服务器）
1. `wget https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v<ver>/kaup-linux-amd64`
2. `chmod +x kaup-linux-amd64 && sudo mv kaup-linux-amd64 /opt/kaup/kaup`
3. `hermes mcp add kaup-cli --command /opt/kaup/kaup`（onboarding 已含）
4. 升级：重复 1-2（替换二进制），Hermes 重启即用新版

## 决策依据

- ADR-0026（D2 三仓体系 + D7 内置 updater + D9 kaup-artifacts 跨仓发布）
- 首次安装下载链接：kaup-design 仓 ONBOARDING.md / 下载页指向本仓 Releases

## 仓维护

- 不要在本仓 commit 二进制（.dmg/.msi/.AppImage）——它们进 Releases attachments，不进 git
- Releases 由 kaup-frontend CI 跨仓 push（KAUPT_ARTIFACTS_TOKEN = GitHub App token，W5-4 配）
- 历史 Release 保留（updater 回滚 + 客户端版本溯源）
