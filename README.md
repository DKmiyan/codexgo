# codexgo 下载：稳定版与预览版

稳定版：[0.1.11](https://github.com/amphiscope/codexgo-releases/releases/tag/v0.1.11)。

## 预览版 0.1.12-preview.1

[发布说明与全部文件](https://github.com/amphiscope/codexgo-releases/releases/tag/v0.1.12-preview.1) · [SHA256SUMS](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.1/SHA256SUMS)

- [darwin_arm64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.1/codexgo_0.1.12-preview.1_darwin_arm64.tar.gz)
- [darwin_amd64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.1/codexgo_0.1.12-preview.1_darwin_amd64.tar.gz)
- [linux_amd64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.1/codexgo_0.1.12-preview.1_linux_amd64.tar.gz)
- [linux_arm64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.1/codexgo_0.1.12-preview.1_linux_arm64.tar.gz)

这是预览版，部分人工验收仍在进行。请校验 SHA256 后解压到独立目录，保留稳定版；当前任务结束并完全退出桌面后，从新路径运行 `codexgo launch`。预览版不通过稳定版自动更新安装。更新主程序不会自动更新旧 Pi／CodeBuddy 适配器，已有会话固定适配器版本；请在插件设置检查，手工配置先备份。

# codexgo

在原版 Codex 桌面版里使用原生 harness。此仓库提供安装包下载与使用说明。

当前下载版本：**0.1.11**。

## 从安装包开始使用

最近的公开安装包是 [v0.1.11](https://github.com/DKmiyan/codexgo/releases/tag/v0.1.11)。不需要源码仓库权限或 Go。下载 Assets 中对应平台的 `codexgo_*.tar.gz` 与 `SHA256SUMS`；GitHub 自动显示的 **Source code** 不是安装包。

本文命令对应 v0.1.11，已包含安全停止握手、事务安装和显式 Bash/zsh 参数。旧 v0.1.10 不具备这些保证；升级旧 daemon 的限制见第 5 节。此版本已通过自动化回归，真实桌面、账号、远端和 iPhone 的完整验收仍在进行。

## 1. 选择并校验安装包

在实际运行 codexgo 的机器上执行 `uname -s` 和 `uname -m`：

| 系统 | CPU | 包名后缀 |
| --- | --- | --- |
| macOS | Apple Silicon / arm64 | `darwin_arm64.tar.gz` |
| macOS | Intel / x86_64 | `darwin_amd64.tar.gz` |
| Linux，包括 WSL2 | x86_64 | `linux_amd64.tar.gz` |
| Linux，包括 WSL2 | aarch64 / arm64 | `linux_arm64.tar.gz` |

以下脚本在新建的临时目录下载、校验并解压所选平台，任何一步失败都会停止。它不会启动或安装程序。

```sh
(
  set -eu
  release_version=0.1.11
  case "$(uname -s)" in
    Darwin) release_os=darwin ;;
    Linux) release_os=linux ;;
    *) echo '只提供 macOS / Linux 安装包' >&2; exit 1 ;;
  esac
  case "$(uname -m)" in
    arm64|aarch64) release_arch=arm64 ;;
    x86_64|amd64) release_arch=amd64 ;;
    *) echo '不支持此 CPU 架构' >&2; exit 1 ;;
  esac
  release_dir=$(mktemp -d)
  cd "$release_dir"
  release_asset="codexgo_${release_version}_${release_os}_${release_arch}.tar.gz"
  release_url="https://github.com/DKmiyan/codexgo/releases/download/v${release_version}"
  curl --fail --location --proto '=https' --tlsv1.2 -O "$release_url/$release_asset"
  curl --fail --location --proto '=https' --tlsv1.2 -O "$release_url/SHA256SUMS"
  awk -v asset="$release_asset" '$2 == asset { print }' SHA256SUMS > selected.sha256
  test "$(wc -l < selected.sha256 | tr -d ' ')" = 1
  if command -v sha256sum >/dev/null 2>&1; then
    sha256sum -c selected.sha256
  else
    shasum -a 256 -c selected.sha256
  fi
  tar -tzf "$release_asset"
  tar -xzf "$release_asset" codexgo
  printf '已校验并解压：%s/codexgo\n' "$release_dir"
)
```

应看到校验结果 `OK`，包内为单个 `codexgo` 程序。保存最后打印的绝对路径，下面用 `/path/to/download/codexgo` 代表它。校验失败时重新下载，不继续安装。安装包与校验文件应来自同一版本。

## 2. Mac 本机

准备已登录的 Codex 桌面版，以及要使用的原生 harness CLI。Claude 的本机登录、Pi 的模型配置、CodeBuddy 的账号都由各自 CLI 管理；codexgo 不替你复制账号材料。

将已校验的程序放在你选择的启动位置。初次安装例如：

```sh
mkdir -p "$HOME/.local/bin"
install -m 755 /path/to/download/codexgo "$HOME/.local/bin/codexgo"
"$HOME/.local/bin/codexgo" version
"$HOME/.local/bin/codexgo" doctor
```

如果该位置已有程序，按下面的升级步骤先保留旧副本。无需修改全局 PATH，也不用修改 Codex 应用包。`doctor` 的路径检查不代表账号已经登录或模型已经获授权；分别在原生 CLI 完成登录并确认可用。

默认桌面应用为 `/Applications/ChatGPT.app`。先等待任务完成，用 Cmd+Q 退出桌面版，再从所选路径启动：

```sh
"$HOME/.local/bin/codexgo" launch
```

应用在其他位置时使用 `launch --app /absolute/path/to/ChatGPT.app`。已有 Codex 会话保留原 harness；要用其他 harness，先新建该 harness 的会话。之后可在同一 harness 内换模型。

Claude 查找失败时，在 `~/.codexgo/config.json` 合并 `claude_bin` 为真实可执行文件的绝对路径。保留已有其他字段，避免指向会注入额外会话参数的包装脚本。

### 界面与 Pi / CodeBuddy 适配器

要显示分组模型菜单、运行计时、插件设置与会话菜单，在 `~/.codexgo/config.json` 中合并 `"renderer_skin": true`，保留已有其他字段，再完全退出并重启。它启用仅本机可访问的调试端口；关闭该字段并重启可停用皮肤。

首次接入 Pi / CodeBuddy：点击顶部齿轮「插件设置」（没有顶部入口时，从输入栏 `⋯` 进入），核对主机后选择「启用适配器」。这只启用 codexgo 自带适配器；原生 CLI 与登录仍由各工具管理。Claude 为内置适配器。已有受管理适配器可以在同一面板更新；已有会话保留所用适配器版本，新开测试会话才能验证新版本。

**从旧版升级的手工注册：** 若面板显示 `unmanaged`，说明 `~/.codexgo/harnesses/pi/manifest.json` 或 `codebuddy/manifest.json` 是旧的手工配置，新面板不会覆盖它。先保留配置备份，检查其中的 `exec` 绝对路径：如果它指向实际更新的 codexgo 文件，完全退出重启即可使用新程序；若指向另一份旧程序，需要同步更新那个路径或把 `exec` 改成新安装路径，保留其他字段。手工注册可继续使用，不必为了测试迁移配置。

需要改用面板管理时，先退出桌面版并结束相关 CLI 会话，将该插件整个目录移到 `harnesses` 之外保留备份，再启动新版，在设置里启用同名适配器。定制参数不会自动迁移，确认需求后再操作；不要删除原生 CLI 的账号与会话目录。

输入栏保留原生上下文圆环；累计 token、缓存和费用在 `⋯` 的用量面板中查看。终端交接也从 `⋯` 进入：等待会话空闲后交接，结束终端 CLI 后明确返回桌面，重新打开该会话读取最新历史。


## 3. Linux / WSL2 远端

Windows 远端使用 WSL2 内的 Linux 用户、项目与 CLI。先配置 SSH；[Microsoft WSL 文档](https://learn.microsoft.com/en-us/windows/wsl/install) 说明 WSL2 的系统要求。无需取得源码才能部署；SSH 应进入 WSL2 的 Linux 用户环境，而非 Windows PowerShell。

在目标 Linux 用户下，按各工具的官方说明安装并登录原生 CLI：

- [Codex CLI](https://github.com/openai/codex#installing-and-running-codex-cli)
- [Claude Code](https://code.claude.com/docs/en/setup)
- [Pi](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent)
- [CodeBuddy CLI](https://www.codebuddy.ai/cli)

只安装需要的 harness。本机登录成功不等于远端成功。先确认从 Mac 使用既有 SSH 别名（下文为 `devbox`）能非交互登录；这一步由用户执行：

```sh
ssh -o BatchMode=yes devbox 'uname -s; uname -m; command -v codex; command -v claude'
```

必须进入 Linux shell。先在桌面版原生 SSH 连接入口打开远端项目，确认原版 Codex 通路可用，再接入 codexgo。

在远端执行第 1 节下载脚本，选择结果应为 `linux_*`。没有外网时，可在能下载的机器取得 Linux 包和对应校验文件，通过已有 SSH/SCP 传入该用户目录，再在远端校验、解压。不要从 Mac 复制 Darwin 二进制到 Linux。

**v0.1.11 事务安装：** 按目标用户实际使用的 shell 显式选择：

```sh
/path/to/download/codexgo remote install --shell bash
# 使用 zsh 的目标用户改为：
/path/to/download/codexgo remote install --shell zsh
```

Bash 写 `~/.bashrc`，zsh 写 `${ZDOTDIR:-$HOME}/.zshenv`。自定义启动文件可传 `--rc /absolute/path`，卸载时必须使用相同 shell 和 RC 参数。未知 shell、符号链接、损坏守卫或冲突的恢复记录会明确报错，不能当成安装成功。

安装落点为 `~/.codexgo/bin/codex`。SSH 守卫设置 `CODEX_INSTALL_DIR`；普通终端中的原版 CLI 保留。若原版 Codex 或 Claude 不在默认位置，在**远端** `~/.codexgo/config.json` 合并：

```json
{
  "real_codex": "/absolute/path/to/native/codex",
  "claude_bin": "/absolute/path/to/native/claude"
}
```

`real_codex` 绝不能指向 `~/.codexgo/bin/codex`，否则递归调用自身。重新连接桌面远端项目；检查项目路径、harness 和会话所属主机。`remote status` 当前只报告路径是否存在，socket 存在不等于 daemon 健康、账号可用或任务成功。

## 4. 验收与排障

先做不产生模型费用的检查：版本、`doctor`、原生 CLI 登录、SSH 登录、模型目录、打开已有历史。需要验证在线模型时，由用户主动发送测试消息，并检查是否仍是同一会话、是否能正常结束。

| 问题 | 检查 |
| --- | --- |
| 下载 404 | 是否选择已经发布的版本和正确包名；勿把 main 功能当作现有 Release |
| 无法执行 | OS/架构是否匹配、可执行权限是否保留；Mac 系统拦截按系统提示核验来源 |
| 有 CLI 但没有模型 | 在目标主机原生 CLI 检查登录、账号权限和模型配置 |
| SSH 可用但桌面接入失败 | 非交互 shell 是否读取守卫；启动文件是否向 stdout 打印欢迎文字 |
| 远端主机离线 | 检查主机睡眠、WSL 生命周期、网络和 SSH；不改成本机续写 |
| 会话被终端占用 | 先结束该终端会话的交互进程；不要同时续写 |
| 升级后版本仍旧 | 当前进程仍用旧二进制；待任务完成后按下一节重连 |

用实际安装路径运行 `codexgo logs` 定位本机日志，远端日志在该用户 `~/.codexgo/logs/`。v0.1.11 默认记录摘要，只有显式 capture 才采集原始 RPC；v0.1.10 的旧日志可能包含正文。分享前检查并删除对话、凭据和私人路径，不直接上传整个日志目录。

## 5. 升级、回退与卸载

**Mac：** 下载并校验新包，等待任务结束并退出桌面版。把旧启动文件复制为带旧版本号的备份，再用新包替换同一路径。执行 `version`，从这个路径重启。回退时退出后换回旧文件。仅想回到原版 Codex，退出后从 Dock 启动即可。

**远端 v0.1.11 新协议：** 新安装器会备份并事务替换磁盘文件，不主动停止正在运行的 daemon。任务完成后断开所有桌面连接，再由已校验的新程序执行：

```sh
/path/to/download/codexgo remote stop
```

成功后重新连接，由新二进制启动 daemon。忙碌、旧协议、身份无法验证或仍在清理时会拒绝/报错；保留错误信息并处理原因，不能把路径存在当作成功，也不要根据 PID 文件直接发送信号。旧 v0.1.10 daemon 不支持新停止握手，需要等待任务结束，按已确认的旧进程退出方法处理；本指南不提供猜测 PID 的强制停止命令。

回退新协议安装时，先正常停止空闲 daemon，再用已校验的旧安装包重新运行 `remote install`。只有支持新事务安装的版本具有新恢复保证；不要宣称回退到 v0.1.10 后仍有同等保护。

卸载前等待任务结束、断开桌面连接。在目标主机运行：

```sh
"$HOME/.codexgo/bin/codex" remote uninstall
```

若安装时指定了 `--shell` / `--rc`，卸载时带上相同参数。卸载移除接入守卫和 codexgo 安装文件，保留原生 CLI、账号、会话、日志和备份。v0.1.11 遇到无法验证的活动 daemon 会拒绝卸载。不要手工清空 `~/.codexgo`；其中可能有会话登记、恢复文件和回收站。

电脑休眠、WSL 关闭和主机重启都会中断运行中的任务；已保存历史与继续运行是两回事。iPhone Remote 对外部 harness 历史的兼容仍待实机验证，当前不承诺手机支持。


## Credits and trademarks

- The Claude logo used by the renderer skin comes from
  [simple-icons](https://simpleicons.org/) (CC0).
- The π logo comes from the pi.dev favicon (MIT).
- Codex, ChatGPT and OpenAI are trademarks of OpenAI. Claude and Claude Code are
  trademarks of Anthropic. Pi is the property of its authors. All other marks
  belong to their respective owners.
- **codexgo is an independent personal project. It is not affiliated with,
  endorsed by, or sponsored by OpenAI or Anthropic.**

## License

The binaries published here are free to use, for any purpose, at your own risk,
with no warranty of any kind. The source code is not public yet and all rights
to it are reserved; no open-source license is granted at this time. Please link
to this page rather than re-hosting the binaries, so people always get the
current release. If licensing matters for your use, open an issue and ask.
