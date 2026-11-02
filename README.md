# kaup-artifacts

kaup 官方更新源 —— 提供 kaup desktop 安装包与 CLI standalone ZIP 的稳定下载点，
以及 desktop auto-updater 与 CLI `kaup upgrade` 所需的更新清单（`latest.json`）。

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
- 产物（同 D12 裁剪）：`kaup-v<version>-{darwin-arm64,windows-amd64}.zip`。
  ZIP 内含 Nuitka standalone runtime、stable launcher 与 `install.json`；不再发布可单独执行的
  raw onefile 资产。
- Linux 需求不在 Release 提供，且 v0.5.3 standalone 契约只覆盖 macOS arm64 与 Windows amd64。
- 同时发布 `latest.json` + `latest.json.sig`。平台条目除 `url`/`sha256` 外还冻结
  `artifact_kind`、`layout_version`、`build_id`、`payload_tree_sha256` 与
  `launcher_protocol_version`；**无自动后台更新**，升级由 IT / 用户显式触发；
  **发布固定不标 latest**（见下「仓库维护」）

## 安装

### Desktop

前往 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载与本机平台匹配的安装包，按系统提示安装即可。首次启动后，客户端会按内置的 updater 公钥定期检查本仓 Releases。

### CLI

> **Transitional status (v0.5.3):** the current `install.sh` / `install.ps1` still implement
> the retired raw-onefile bootstrap. The release workflow fails closed while either script
> carries its `TRANSITIONAL:` marker, so these scripts must not be used to publish or install
> a production v0.5.3 standalone release. The standalone clean-bootstrap installer is owned by
> the installer child and must land first.

The historical fixed-tag commands remain the eventual stable entry point once that installer
child lands:

macOS：

```bash
curl -fsSLO https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-latest/install.sh
sh install.sh
```

Windows（PowerShell）：

```powershell
curl.exe -fsSLO https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-latest/install.ps1
powershell -ExecutionPolicy Bypass -File install.ps1
```

两条 URL 指向**固定 tag `kaup-cli-latest`**：脚本由 `kaup-core` 仓 `install/` 目录来，随每个版本
release 发到本仓；固定 tag 上的那两个由 `maintain-fixed-tag` 在**正式发布**时同步。URL 长期不变
——脚本本身**与版本无关**，它默认就从该固定 tag 读 `latest.json`。

The rewritten installer must install the stable launcher at `~/.kaup/bin/kaup`
(Windows: `%USERPROFILE%\.kaup\bin\kaup.exe`), extract the ZIP into the immutable
`~/.kaup/versions/<build-id>/` transaction, and only then switch `~/.kaup/current`.
First installation still relies on TLS plus the signed manifest/ZIP SHA256; it must not claim
that the installer itself verified a minisign signature.

> ⚠ **固定 tag 尚未创建（截至 2026-10-16）**：`kaup-cli-latest` / `kaup-desktop-latest` 由
> `maintain-fixed-tag` 在**正式发布**时创建并推进，而本波尚未发过正式版 ⇒ 上面两条 URL
> **今天会 404**。首次正式发布后即生效。
>
> 在此之前要装 / 验**预发布版**，请取**版本 tag** 的脚本并显式指定取数点：
>
> ```bash
> TAG=kaup-cli-v0.5.0-rc.8
> curl -fsSLO "https://github.com/kaup-ai/kaup-artifacts/releases/download/$TAG/install.sh"
> KAUP_INSTALL_BASE_URL="https://github.com/kaup-ai/kaup-artifacts/releases/download/$TAG" sh install.sh
> ```

> ⚠ 自升级（`kaup upgrade`）**只替换 `~/.kaup/bin`（= `$KAUP_HOME/bin`）内的二进制**。
> 装到该处之外（例如此前文档所示的 `/opt/kaup/kaup` + `/usr/local/bin` 符号链接）会被
> 判为「不在安装根内」而**拒绝升级**——请用上面的安装脚本重装到安装根。

The ZIP is an installation transaction input, not a replacement executable. Do not unpack it
over `~/.kaup/bin` or move the runtime there manually: upstream transaction and recovery
semantics require staging, payload-tree verification, the stable launcher, and an atomic
`current` switch. Historical releases may still contain raw onefile assets and remain available
for rollback evidence, but they are not the v0.5.3 distribution contract.

## 升级

### Desktop

无需手动操作。客户端启动时静默检查更新，发现新版后在设置菜单「关于」面板提示（不打断工作）；用户确认下载并验签通过后，经「立即重启」完成替换。如需强制升级，从 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载最新安装包重装即可。

### CLI

An existing onefile installation cannot use its old self-updater to replace itself with the
standalone ZIP. After the installer and runtime children land, run the updated official
installer exactly once to migrate. Subsequent A-to-B upgrades will use the stable launcher and
immutable version transaction; `kaup upgrade` will verify the signed manifest and ZIP bytes
before switching `current`.

## 验签

### Desktop

每个更新包附带 `.sig` 签名文件。Tauri 客户端内置发布方公钥，下载完成后自动验签：签名不匹配则拒绝替换，从源头阻断伪造更新。fork / 镜像本仓的产物因无对应私钥，无法生成客户端接受的签名。

如需手动验签（CI 或审计场景）：

```bash
# latest.json 示例（实际字段以 release 为准）
# 客户端只信任内置公钥对应的签名；手动验签需要获取发布方公钥
```

### CLI

CLI releases publish a minisign-signed `latest.json`. Each platform entry binds the final ZIP
SHA256 to the archive's `install.json` build-id and payload-tree digest. Once the runtime child
lands, `kaup upgrade` will verify the manifest signature and downloaded bytes before changing
the active version. The ZIP and extracted binaries are intentionally unsigned at the
OS-publisher layer for v0.5.3; Gatekeeper and SmartScreen warnings remain possible.

## 仓库维护

- Releases 由上游 CI 在 tag 推送时自动创建：CLI 走 `kaup-core/.github/workflows/release.yml`，desktop 走 `kaup-frontend/.github/workflows/release.yml`；请勿在本仓直接 commit 二进制或 `latest.json`
- **desktop updater 走固定 tag，不依赖 `releases/latest`**（R3 / 2026-09-07 迁移）：endpoint 为 `releases/download/kaup-desktop-latest/latest.json`。CLI 发布 CI 仍固定 `make_latest: false`，使 `releases/latest` 指向 desktop 供人工浏览——这是浏览便利约定，已不是 updater 的依赖路径
- 历史 Release 不删除，便于 updater 回滚与版本溯源
- 仓库无源码，仅含文档与配置：`README.md` / `LICENSE` / `AGENTS.md` / `.gitignore` / `docs/agents/`

## License

本仓及 Releases 分发的所有产物均为 Dalian Kaup Technology Co., Ltd. 的专有软件，版权归其所有，详见 [LICENSE](./LICENSE)。第三方依赖归因已内嵌于各产物（CLI standalone ZIP / desktop 安装包）中。
