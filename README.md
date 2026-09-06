# kaup-artifacts

kaup 官方更新源 —— 提供 kaup desktop 安装包与 CLI 二进制的稳定下载点，以及 Tauri auto-updater 所需的更新清单（`latest.json`）。

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
- 产物（同 D12 裁剪）：单文件二进制 `kaup-<os>-<arch>`（macOS 无后缀，Windows 为 `.exe`），仅 macOS + Windows
- Linux 需求不在 Release 提供：IT 在目标机经源码构建（`kaup-core` `scripts/build-cli.sh`，uname 自适应）
- 不发布 `latest.json`：服务端工具，IT deliberate 升级，无 auto-updater；**发布固定不标 latest**（见下「仓库维护」）

## 安装

### Desktop

前往 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载与本机平台匹配的安装包，按系统提示安装即可。首次启动后，客户端会按内置的 updater 公钥定期检查本仓 Releases。

### CLI

以 macOS ARM64 为例（其余平台命令见下表；Linux 经源码构建，不走 Release）：

```bash
wget https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v<ver>/kaup-darwin-arm64
chmod +x kaup-darwin-arm64
sudo mv kaup-darwin-arm64 /opt/kaup/kaup
sudo ln -s /opt/kaup/kaup /usr/local/bin/kaup
```

按平台分别执行：

| 平台 | 二进制 | 安装路径 | 后续步骤 |
|------|--------|----------|----------|
| macOS | `kaup-darwin-<arch>` | `/usr/local/bin/kaup` | 同上；首次运行需解除 Gatekeeper 隔离：`sudo xattr -d com.apple.quarantine /opt/kaup/kaup` |
| Windows | `kaup-windows-amd64.exe` | `C:\Program Files\kaup\kaup.exe` | 将该目录加入系统 PATH |

## 升级

### Desktop

无需手动操作。客户端启动时静默检查更新，发现新版后在设置菜单「关于」面板提示（不打断工作）；用户确认下载并验签通过后，经「立即重启」完成替换。如需强制升级，从 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载最新安装包重装即可。

### CLI

按上表重新执行"下载 → 移动 → 建符号链接"三步，覆盖旧二进制后重启调用方即生效。

## 验签

### Desktop

每个更新包附带 `.sig` 签名文件。Tauri 客户端内置发布方公钥，下载完成后自动验签：签名不匹配则拒绝替换，从源头阻断伪造更新。fork / 镜像本仓的产物因无对应私钥，无法生成客户端接受的签名。

如需手动验签（CI 或审计场景）：

```bash
# latest.json 示例（实际字段以 release 为准）
# 客户端只信任内置公钥对应的签名；手动验签需要获取发布方公钥
```

### CLI

当前阶段建议 IT 在部署前执行 SHA256 checksum 核对（Release 页每项资产下展示）。后续将提供 cosign / GPG 签名，部署脚本应 verify 后再替换。

## 仓库维护

- Releases 由上游 CI 在 tag 推送时自动创建，请勿在本仓直接 commit 二进制或 `latest.json`
- **`releases/latest` 指针专属 desktop**（2026-09-06 约定，设置菜单增强波次 D13）：desktop updater endpoint 为 `releases/latest/download/latest.json`，依赖 latest 指针始终指向 desktop release——CLI 发布 CI 固定 `make_latest: false`，任何新 release 类目上线前必须先确认不抢占 latest 指针
- 历史 Release 不删除，便于 updater 回滚与版本溯源
- 仓库仅含本 README 与 `.gitignore`，无其他源码

## License

本仓及 Releases 分发的所有产物均为 Dalian Kaup Technology Co., Ltd. 的专有软件，版权归其所有，详见 [LICENSE](./LICENSE)。第三方依赖归因已内嵌于各产物（CLI exe / desktop 安装包）中。