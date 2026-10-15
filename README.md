# kaup-artifacts

kaup 官方更新源 —— 提供 kaup desktop 安装包与 CLI 二进制的稳定下载点，以及 desktop auto-updater 与 CLI `kaup upgrade` 所需的更新清单（`latest.json`）。

## Releases

本仓只承载 [GitHub Releases](https://github.com/kaup-ai/kaup-artifacts/releases)；源代码、二进制均不进 git。

### Desktop

面向最终业务人员使用的桌面客户端，每人一台。

- Release tag：`kaup-desktop-v<version>`
- 产物（2026-09-06 起仅两平台，设置菜单增强波次 D12 裁剪）：macOS `.dmg` + `.app.tar.gz` / Windows `.msi` + `.nsis`，updater 产物均附 `.sig` 签名
- 同时发布 `latest.json`（Tauri updater 清单：各平台当前版本号、签名与下载 URL）

### CLI

面向共享服务器，由 IT 统一部署，一台一公司。

- Release tag：`kaup-cli-v<version>`
- 产物（同 D12 裁剪）：单文件二进制 `kaup-v<version>-<os>-<arch>`（macOS 无后缀，Windows 为 `.exe`），仅 macOS（`darwin-arm64`）+ Windows（`windows-amd64`）
- Linux 需求不在 Release 提供：IT 在目标机经源码构建（`kaup-core` `scripts/build-cli.sh`，uname 自适应）
- 同时发布 `latest.json` + `latest.json.sig`（minisign 签名，A4/2026-09-07）供 `kaup upgrade` 拉取；**无自动后台更新**，升级由 IT / 用户显式触发；**发布固定不标 latest**（见下「仓库维护」）

## 安装

### Desktop

前往 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载与本机平台匹配的安装包，按系统提示安装即可。首次启动后，客户端会按内置的 updater 公钥定期检查本仓 Releases。

### CLI

**推荐：官方安装脚本** —— 把 CLI 装到自升级安装根 `~/.kaup/bin`（即 `$KAUP_HOME/bin`）。
脚本取自 `kaup-core` 仓 `install/` 目录（`install.sh` / `install.ps1`）：

- macOS：`sh install.sh`（Linux 无 Release，见下「手动安装」）
- Windows（PowerShell）：`powershell -ExecutionPolicy Bypass -File install.ps1`

脚本拉固定 tag `kaup-cli-latest` 的 `latest.json` → **校 sha256** → 落
`~/.kaup/bin/kaup`（Windows：`%USERPROFILE%\.kaup\bin\kaup.exe`）→ 打印下一步
（`~/.kaup/bin/kaup onboard`）。脚本**不做 minisign 验签**：首次安装只信任 TLS +
清单 sha256；后续升级由 `kaup upgrade` 走完整 minisign 链（见下「验签」）。

> ⚠ 自升级（`kaup upgrade`）**只替换 `~/.kaup/bin`（= `$KAUP_HOME/bin`）内的二进制**。
> 装到该处之外（例如此前文档所示的 `/opt/kaup/kaup` + `/usr/local/bin` 符号链接）会被
> 判为「不在安装根内」而**拒绝升级**——请用上面的安装脚本重装到安装根。

**手动安装（源码 / Linux 构建产物，或系统级放置；不参与自升级）**：

Linux 不提供 Release（经源码构建：`kaup-core` 仓 `scripts/build-cli.sh`）。手工放置
（例如此前的系统级 `/opt/kaup` + 符号链接 `/usr/local/bin/kaup`）**不参与自升级**——
`kaup upgrade` 只替换安装根 `~/.kaup/bin` 内的二进制：

```bash
wget https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v<ver>/kaup-v<ver>-darwin-arm64
chmod +x kaup-v<ver>-darwin-arm64
sudo mv kaup-v<ver>-darwin-arm64 /opt/kaup/kaup
sudo ln -s /opt/kaup/kaup /usr/local/bin/kaup
```

要参与自升级，请改用上面的安装脚本（落 `~/.kaup/bin`）。

| 平台 | 二进制 | 手动安装路径（不参与自升级） | 后续步骤 |
|------|--------|----------|----------|
| macOS（`darwin-arm64`） | `kaup-v<ver>-darwin-arm64` | `/usr/local/bin/kaup` | 首次运行需解除 Gatekeeper 隔离：`sudo xattr -d com.apple.quarantine /opt/kaup/kaup` |
| Windows（`windows-amd64`） | `kaup-v<ver>-windows-amd64.exe` | `C:\Program Files\kaup\kaup.exe` | 将该目录加入系统 PATH |

## 升级

### Desktop

无需手动操作。客户端启动时静默检查更新，发现新版后在设置菜单「关于」面板提示（不打断工作）；用户确认下载并验签通过后，经「立即重启」完成替换。如需强制升级，从 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载最新安装包重装即可。

### CLI

发布版可直接运行 `kaup upgrade`：从固定 tag `kaup-cli-latest` 拉 `latest.json` + `.sig`，minisign 验签通过后替换自身。手动「下载 → 移动 → 建符号链接」三步仍适用于源码 / Linux 构建的产物。

## 验签

### Desktop

每个更新包附带 `.sig` 签名文件。Tauri 客户端内置发布方公钥，下载完成后自动验签：签名不匹配则拒绝替换，从源头阻断伪造更新。fork / 镜像本仓的产物因无对应私钥，无法生成客户端接受的签名。

如需手动验签（CI 或审计场景）：

```bash
# latest.json 示例（实际字段以 release 为准）
# 客户端只信任内置公钥对应的签名；手动验签需要获取发布方公钥
```

### CLI

CLI 发布附带 minisign 签名的 `latest.json`（内含各平台 SHA256），`kaup upgrade` 自动验签通过后才替换。IT 手动部署时可据此核对资产完整性；源码构建产物不在 Release 链路内，完整性由构建方负责。

## 仓库维护

- Releases 由上游 CI 在 tag 推送时自动创建：CLI 走 `kaup-core/.github/workflows/release.yml`，desktop 走 `kaup-frontend/.github/workflows/release.yml`；请勿在本仓直接 commit 二进制或 `latest.json`
- **desktop updater 走固定 tag，不依赖 `releases/latest`**（R3 / 2026-09-07 迁移）：endpoint 为 `releases/download/kaup-desktop-latest/latest.json`。CLI 发布 CI 仍固定 `make_latest: false`，使 `releases/latest` 指向 desktop 供人工浏览——这是浏览便利约定，已不是 updater 的依赖路径
- 历史 Release 不删除，便于 updater 回滚与版本溯源
- 仓库无源码，仅含文档与配置：`README.md` / `LICENSE` / `AGENTS.md` / `.gitignore` / `docs/agents/`

## License

本仓及 Releases 分发的所有产物均为 Dalian Kaup Technology Co., Ltd. 的专有软件，版权归其所有，详见 [LICENSE](./LICENSE)。第三方依赖归因已内嵌于各产物（CLI exe / desktop 安装包）中。