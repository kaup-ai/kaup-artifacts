# kaup-artifacts

kaup 官方更新源 —— 提供 kaup desktop 安装包与 CLI 二进制的稳定下载点，以及 Tauri auto-updater 所需的更新清单（`latest.json`）。

## Releases

本仓只承载 [GitHub Releases](https://github.com/kaup-ai/kaup-artifacts/releases)；源代码、二进制均不进 git。

### Desktop

面向最终业务人员使用的桌面客户端，每人一台。

- Release tag：`kaup-desktop-v<version>`
- 产物：macOS `.dmg` / Ubuntu `.AppImage` + `.deb` / Windows `.msi` + `.nsis`，均附 `.sig` 签名
- 同时发布 `latest.json`（Tauri updater 清单：各平台当前版本号、签名与下载 URL）

### CLI

面向共享服务器，由 IT 统一部署，一台一公司。

- Release tag：`kaup-cli-v<version>`
- 产物：单文件二进制 `kaup-<os>-<arch>`（Linux/macOS 无后缀，Windows 为 `.exe`）
- 不发布 `latest.json`：服务端工具，IT deliberate 升级，无 auto-updater

## 安装

### Desktop

前往 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载与本机平台匹配的安装包，按系统提示安装即可。首次启动后，客户端会按内置的 updater 公钥定期检查本仓 Releases。

### CLI

以 Linux AMD64 为例（其余平台命令见下表）：

```bash
wget https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v<ver>/kaup-linux-amd64
chmod +x kaup-linux-amd64
sudo mv kaup-linux-amd64 /opt/kaup/kaup
sudo ln -s /opt/kaup/kaup /usr/local/bin/kaup
```

按平台分别执行：

| 平台 | 二进制 | 安装路径 | 后续步骤 |
|------|--------|----------|----------|
| Linux | `kaup-linux-amd64` | `/usr/local/bin/kaup` | `sudo ln -s /opt/kaup/kaup /usr/local/bin/kaup` |
| macOS | `kaup-darwin-<arch>` | `/usr/local/bin/kaup` | 同上；首次运行需解除 Gatekeeper 隔离：`sudo xattr -d com.apple.quarantine /opt/kaup/kaup` |
| Windows | `kaup-windows-amd64.exe` | `C:\Program Files\kaup\kaup.exe` | 将该目录加入系统 PATH |

## 升级

### Desktop

无需手动操作。客户端启动时检测新版本，下载并验签通过后自动替换；下次启动即生效。如需强制升级，从 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载最新安装包重装即可。

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
- 历史 Release 不删除，便于 updater 回滚与版本溯源
- 仓库仅含本 README 与 `.gitignore`，无其他源码