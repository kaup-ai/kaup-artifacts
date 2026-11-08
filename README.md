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
- 同版本另含 `install.sh`、`install.ps1`、`latest.json` 与 `latest.json.sig`；稳定无版本
  CDN 副本位于 `channels/cli/install.{sh,ps1}`，不可变审计副本位于
  `releases/cli/<version>/install.{sh,ps1}`。
- Linux 需求不在 Release 提供，且 v0.5.3 standalone 契约只覆盖 macOS arm64 与 Windows amd64。
- 同时发布 `latest.json` + `latest.json.sig`。平台条目除 `url`/`sha256` 外还冻结
  `artifact_kind`、`layout_version`、`build_id`、`payload_tree_sha256` 与
  `launcher_protocol_version`；**无自动后台更新**，升级由 IT / 用户显式触发；
  **发布固定不标 latest**（见下「仓库维护」）

## 安装

### Desktop

CDN 是 Desktop 的公开主下载源；GitHub Releases 是独立镜像与 fallback。
平台安装包的当前直链、大小与 SHA256 以
[`channels/downloads.json`](https://downloads.kaup-ai.com/channels/downloads.json)
为准。也可以前往
[Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop)
从 GitHub 镜像下载。安装包继续使用现有 Tauri updater 签名链。

### CLI

The installers are the primary customer entry point. They consume the same
standalone ZIP as `kaup upgrade`; the ZIP is an advanced/offline asset, not a
program to run directly.

macOS convenience command:

```bash
curl -fsSL https://downloads.kaup-ai.com/channels/cli/install.sh | sh
```

Windows PowerShell (download and execute by file, preserving process exit
semantics):

```powershell
Invoke-WebRequest https://downloads.kaup-ai.com/channels/cli/install.ps1 -OutFile install.ps1
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\install.ps1
```

`ExecutionPolicy Bypass` is process-scoped so the downloaded script can run on
a fresh Windows client whose policy is `Restricted`; it does not change the
machine policy.

`channels/cli/` is a short-TTL CDN prefix. The stable scripts are overwritten
only from the exact versioned bytes at
`releases/cli/<version>/install.{sh,ps1}`. Do not use the CDN root: it has
different cache semantics and is reserved for a future human entry page.

For a download -> verify -> run path, compare the stable object with the
versioned GitHub-release copy and with the published checksum:

```bash
VERSION=<version>
BASE=https://downloads.kaup-ai.com
GH=https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v$VERSION
curl -fsSLO "$BASE/channels/cli/install.sh"
curl -fsSL "$GH/install.sh" -o install.sh.github
curl -fsSL "$BASE/checksums/$VERSION/cli/SHA256SUMS" -o SHA256SUMS
expected="$(shasum -a 256 install.sh.github | awk '{print $1}')"
actual="$(shasum -a 256 install.sh | awk '{print $1}')"
test "$expected" = "$actual" || { echo "installer sha256 mismatch" >&2; exit 1; }
test "$expected" = "$(awk '$2 == "install.sh" {print $1}' SHA256SUMS)" ||
  { echo "published checksum mismatch" >&2; exit 1; }
sh ./install.sh
```

PowerShell equivalent:

```powershell
$Version = '<version>'
$Base = 'https://downloads.kaup-ai.com'
$Gh = "https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-v$Version"
Invoke-WebRequest "$Base/channels/cli/install.ps1" -OutFile install.ps1
Invoke-WebRequest "$Gh/install.ps1" -OutFile install.ps1.github
Invoke-WebRequest "$Base/checksums/$Version/cli/SHA256SUMS" -OutFile SHA256SUMS
$expected = (Get-FileHash .\install.ps1.github -Algorithm SHA256).Hash.ToLowerInvariant()
$actual = (Get-FileHash .\install.ps1 -Algorithm SHA256).Hash.ToLowerInvariant()
if ($expected -cne $actual) { throw 'installer sha256 mismatch' }
$published = ((Select-String '  install\.ps1$' SHA256SUMS).Line -split '\s+')[0]
if ($expected -cne $published) { throw 'published checksum mismatch' }
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\install.ps1
```

The versioned `SHA256SUMS` contains both platform ZIPs and both installers.
The same installer hashes are published in each CLI GitHub release note.
`irm | iex` remains possible, but pipes input into the interpreter and can lose
the script process exit code, so the documented primary Windows form uses
`-File`.

The installer defaults to
`https://downloads.kaup-ai.com/channels/cli/latest.json`. The GitHub fixed-tag
scripts remain a transport/audit fallback; to force GitHub-only data access:

```bash
KAUP_INSTALL_BASE_URL=https://github.com/kaup-ai/kaup-artifacts/releases/download/kaup-cli-latest \
  sh install.sh
```

The installer reads either the CDN signature envelope or raw GitHub manifest
over TLS. For the envelope it validates the exact outer schema, strictly
base64-decodes the bounded inner manifest, and applies the normal manifest
schema; it does not treat envelope fields as manifest fields. It then verifies
the ZIP SHA256 and payload tree, extracts into `~/.kaup/versions/<build-id>/`,
installs the stable launcher at `~/.kaup/bin/kaup` (Windows:
`%USERPROFILE%\.kaup\bin\kaup.exe`), and writes `~/.kaup/current` plus
`install-state.json`. First installation trusts TLS plus the inner
manifest-provided ZIP SHA256; decoding the envelope does not verify its
minisign signature, and the installer does not claim that it did.
If a usable stable launcher already exists, the installer delegates to
`kaup upgrade --all`.

> ✅ **固定 tag 已创建（2026-10-31 起）**：`kaup-cli-latest` / `kaup-desktop-latest` 由
> `maintain-fixed-tag` 维护并随正式版推进；`v0.5.1`（2026-10-31）与 `v0.5.2`（2026-11-02）
> 已发布，上面的固定 tag URL 现已可用。
>
> ⚠ 截至 2026-11-08：CDN 的 `channels/downloads.json` 已上线，但
> `channels/cli/install.{sh,ps1}` **尚未发布**（404；由 oss-installer 发布波次上线）。
> CDN 安装命令生效前，请改用固定 tag / 版本 tag 的 GitHub 路径安装：
>
> ```bash
> TAG=kaup-cli-v0.5.2
> curl -fsSLO "https://github.com/kaup-ai/kaup-artifacts/releases/download/$TAG/install.sh"
> KAUP_INSTALL_BASE_URL="https://github.com/kaup-ai/kaup-artifacts/releases/download/$TAG" sh install.sh
> ```

> ⚠ 自升级（`kaup upgrade`）只能从 `~/.kaup/bin`（= `$KAUP_HOME/bin`）内的稳定
> launcher 发起，并原子切换 `~/.kaup/current` 指向的 immutable version；普通升级不会
> 替换运行中的稳定 launcher。装到该处之外（例如此前文档所示的 `/opt/kaup/kaup` +
> `/usr/local/bin` 符号链接）会被判为「不在安装根内」而**拒绝升级**——请用上面的
> 安装脚本重装到安装根。

The ZIP is an installation transaction input, not a replacement executable. Do not unpack it
over `~/.kaup/bin` or move the runtime there manually: upstream transaction and recovery
semantics require staging, payload-tree verification, the stable launcher, and an atomic
`current` switch. Historical releases may still contain raw onefile assets and remain available
for rollback evidence, but they are not the v0.5.3 distribution contract.

## 升级

### Desktop

无需手动操作。客户端启动时静默检查更新，发现新版后在设置菜单「关于」面板提示（不打断工作）；用户确认下载并验签通过后，经「立即重启」完成替换。如需强制升级，从 [Releases 页](https://github.com/kaup-ai/kaup-artifacts/releases?q=kaup-desktop) 下载最新安装包重装即可。

### CLI

An existing onefile installation cannot migrate itself or be overwritten by
the standalone installer. The installer treats the old `bin/kaup[.exe]` as an
unproven launcher and fails closed. A supervised migration must stop the old
daemons and remove or relocate that legacy entry before running the installer;
a single installer rerun does not perform an in-place onefile migration. Later
runs detect the resulting stable launcher and delegate to the CLI transaction.
Subsequent A-to-B upgrades use the stable launcher and immutable version
transaction; `kaup upgrade` verifies the signed manifest and ZIP bytes before
switching `current`.

## 验签

### Desktop

每个更新包附带 `.sig` 签名文件。Tauri 客户端内置发布方公钥，下载完成后自动验签：签名不匹配则拒绝替换，从源头阻断伪造更新。fork / 镜像本仓的产物因无对应私钥，无法生成客户端接受的签名。

如需手动验签（CI 或审计场景）：

```bash
# latest.json 示例（实际字段以 release 为准）
# 客户端只信任内置公钥对应的签名；手动验签需要获取发布方公钥
```

### CLI

CLI releases publish a minisign-signed `latest.json`. Each platform entry binds
the final ZIP SHA256 to the archive's `install.json` build-id and payload-tree
digest. `kaup upgrade` verifies the manifest signature and downloaded bytes
before changing the active version. The ZIP and extracted binaries are
intentionally unsigned at the OS-publisher layer for v0.5.3; Gatekeeper and
SmartScreen warnings remain possible.

## 仓库维护

- Releases 由上游 CI 在 tag 推送时自动创建：CLI 走 `kaup-core/.github/workflows/release.yml`，desktop 走 `kaup-frontend/.github/workflows/release.yml`；请勿在本仓直接 commit 二进制或 `latest.json`
- **desktop updater 走固定 tag，不依赖 `releases/latest`**（R3 / 2026-09-07 迁移）：endpoint 为 `releases/download/kaup-desktop-latest/latest.json`。CLI 发布 CI 仍固定 `make_latest: false`，使 `releases/latest` 指向 desktop 供人工浏览——这是浏览便利约定，已不是 updater 的依赖路径
- 历史 Release 不删除，便于 updater 回滚与版本溯源
- 仓库无源码，仅含文档与配置：`README.md` / `LICENSE` / `AGENTS.md` / `.gitignore` / `docs/agents/`

## License

本仓及 Releases 分发的所有产物均为 Dalian Kaup Technology Co., Ltd. 的专有软件，版权归其所有，详见 [LICENSE](./LICENSE)。第三方依赖归因已内嵌于各产物（CLI standalone ZIP / desktop 安装包）中。
