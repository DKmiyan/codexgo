# codexgo 下载：稳定版与预览版

稳定版：[0.1.11](https://github.com/amphiscope/codexgo-releases/releases/tag/v0.1.11)。

## 预览版 0.1.12-preview.2

[发布说明与全部文件](https://github.com/amphiscope/codexgo-releases/releases/tag/v0.1.12-preview.2) · [SHA256SUMS](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.2/SHA256SUMS)

- [darwin_arm64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.2/codexgo_0.1.12-preview.2_darwin_arm64.tar.gz)
- [darwin_amd64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.2/codexgo_0.1.12-preview.2_darwin_amd64.tar.gz)
- [linux_amd64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.2/codexgo_0.1.12-preview.2_linux_amd64.tar.gz)
- [linux_arm64](https://github.com/amphiscope/codexgo-releases/releases/download/v0.1.12-preview.2/codexgo_0.1.12-preview.2_linux_arm64.tar.gz)

这是预览版，完整人工验收仍在进行。请校验 SHA256 后解压到独立目录，保留稳定版；当前任务结束并完全退出桌面后，从新路径运行 `codexgo launch`。预览版不通过稳定版自动更新安装。更新主程序不会自动更新旧 Pi／CodeBuddy 适配器，已有会话固定适配器版本；请在插件设置检查，手工配置先备份。

本版包含 Claude 动态模型与 Effort、会话状态、历史分页和工具展示修复。远端安装器覆盖接管程序、Pi 回收站、原生恢复默认和 CodeBuddy 中断后的上下文仍在跟进，详见上方发布说明。

# codexgo

在原版 Codex 桌面版里使用原生 harness。此仓库提供安装包下载与使用说明。

以下安装命令默认对应稳定版：**0.1.11**。

## 从安装包开始使用

当前稳定版是 [v0.1.11](https://github.com/amphiscope/codexgo-releases/releases/tag/v0.1.11)。不需要源码仓库权限或 Go。下载 Assets 中对应平台的 `codexgo_*.tar.gz` 与 `SHA256SUMS`；GitHub 自动显示的 **Source code** 不是安装包。

本文命令对应 v0.1.11，已包含安全停止握手、事务安装和显式 Bash/zsh 参数。旧 v0.1.10 不具备这些保证；升级旧 daemon 的限制见第 5 节。此版本已通过自动化回归，真实桌面、账号、远端和 iPhone 的完整验收仍在进行。

**0.1.12-preview.2 预览版：** 从下载页选择同名 prerelease，校验后解压到独立目录并保留稳定版。任务结束、完全退出桌面后，从新路径运行 `codexgo launch`；预览版不会通过稳定版自动更新安装。它包含 Claude 动态模型与 Effort、会话状态、历史分页和工具展示修复，完整人工验收仍在进行。

**预览版已知限制：** 已部署远端的原生 CLI 安装器可能覆盖 codexgo 接管程序；远端重新安装或更新前请先核对路径，不要在活动会话中操作。Pi 原生会话回收站、原生“恢复默认”意图区分、CodeBuddy 中断后的上下文保留仍在调查，不能视为本版已修复。没有独立附件记录时，附件面板返回空列表；聊天历史中的图片不受该列表替代。

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
  release_url="https://github.com/amphiscope/codexgo-releases/releases/download/v${release_version}"
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

**Claude 模型与 Effort（0.1.12-preview.2）：** 模型列表和可用强度来自原生 Claude CLI 目录；先更新原生 CLI，再在插件设置检查目录。新模型是否可用取决于当前账号和原生 CLI 的返回。默认强度显示“继承默认”，只有运行时明确回报后才显示实际档位。目录已成功获取但缓存过期时仍可选择已有模型，设置会标记缓存；失败或目录不完整仍会提示。

**从旧版升级的手工注册：** 若面板显示 `unmanaged`，说明 `~/.codexgo/harnesses/pi/manifest.json` 或 `codebuddy/manifest.json` 是旧的手工配置，新面板不会覆盖它。先保留配置备份，检查其中的 `exec` 绝对路径：如果它指向实际更新的 codexgo 文件，完全退出重启即可使用新程序；若指向另一份旧程序，需要同步更新那个路径或把 `exec` 改成新安装路径，保留其他字段。手工注册可继续使用，不必为了测试迁移配置。

**Pi 启动后仍只显示一个模型（修复包含于 0.1.12-preview.1）：** 原生 Pi 冷启动可能超过旧版 1 秒查询期限，导致菜单只保留 manifest 的旧模型。修复把启动模型发现改为单插件 6 秒、整批 8 秒预算，正常完成即返回；插件设置里的显式「检查」使用同样额度。保持 Pi 扩展注册的模型来源，不通过禁用扩展缩短时间，也不发送聊天请求。升级受管理 Pi 适配器后再用新会话验证；若仍超时，在插件设置查看查询状态，不把回退模型当作完整目录。模型选择器会按当前主机显示「目录待刷新」或「目录未验证」，查询恢复后提示自动消失；已有模型仍可选择。旧 daemon 未提供目录状态时不会猜测其完整性。

**CodeBuddy 模型列表修复（0.1.12-preview.1）：** 安装含此修复的 codexgo 后，在「插件设置」选中正确主机，为受管理的 CodeBuddy 适配器点击「更新」，再「检查」。仅替换主程序或重启不会更新旧的独立适配器文件。手工注册则先备份自己的 manifest，确认 `exec` 指向新版程序，并在 `capabilities` 数组中追加 `"models"`；保留其他自定义字段，不要覆盖整份配置。示例见 `examples/harnesses/codebuddy/manifest.json`。直接的只读检查是 `codexgo harness codebuddy query models`，它不发聊天消息；命令成功不等于 manifest 已启用发现。

新适配器由原生 CodeBuddy 返回本机用户级模型目录，不写死模型 ID，不读取或复制账号文件，也不会创建聊天记录。项目专属的 `models.json` 不进入这个主机级菜单。目录成功但为空时不补造模型；目录查询失败时插件状态会标明未验证或过期，不能把保留的 `default` 入口当成成功枚举。列出模型不代表账号一定有调用权限。已有会话仍保留原适配器版本，更新后的模型选择请在新测试会话检查。


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

### 更新命令与预览版限制

以下命令包含于 0.1.12-preview.1 的代码，但仅处理正式版更新；预览版默认受非正式构建保护，不提供 preview 渠道自动更新。试用 preview 请下载、校验并保存在独立路径，任务结束后从该路径启动；保留原稳定程序便于回退。公开 v0.1.11 不包含这些命令，不要在旧版尝试 `update`，以免转交原生 Codex。无需自建网站、服务器或域名，正式版查询和安装包下载使用公开下载仓库的 GitHub Release。首次公开 curl 安装入口尚未提供，发布前需要单独审查脚本和资产策略；不要猜测 install.sh 下载地址。

```sh
codexgo update --check        # 只查询版本及校验元数据，不改安装
codexgo update --dry-run      # 查看更新计划，不下载程序或修改安装
# 等待任务与待发送消息全部完成，Cmd+Q 退出客户端后：
codexgo update               # 安装最新正式版，校验、备份、原子替换
codexgo launch

# 指定已发布的正式版；较旧版本还须明确加 --allow-downgrade
codexgo update --version 0.1.12
# 出现问题时，先退出客户端再回退；可先 --dry-run 看备份位置
codexgo rollback
```

示例版本号只说明参数形式，不代表该版本已发布。检查和 dry-run 仍需联网读取发布元数据；网络失败会报错，不等于已是最新版。请求只访问固定的公开下载仓库及 GitHub 资产服务，不访问私有源码仓库、不要求用户配置 GitHub token。SHA256 验证下载完整性，信任基础是 HTTPS 和发布账号，并非独立的发布者签名。

`update` 更新当前执行的 `codexgo` 安装路径，拒绝数据目录内的适配器与远端 `codex` 安装。安装目录必须是普通目录；不自动改 PATH、创建系统目录或请求 sudo。下载/校验失败不会换掉旧程序；同一安装不允许并发更新，旧程序和恢复记录保存在目标文件旁的私有更新目录。回退会核对当前安装及备份，遇到用户另行修改的文件会拒绝覆盖。正常回退恢复上次安装前的程序，不是下载任意历史版本。若存在中断记录，rollback 首先只恢复这次事务：文件尚未替换则撤销，已替换且校验匹配则完成登记，不会连续多退一版；根据输出确认磁盘版本后再决定后续操作。旧的手工备份不会自动导入这份回退记录。若首次安装中断、目标程序还不存在，可从另一份已校验的新版程序执行 `rollback --dir /existing/bin` 恢复该目录的事务。回退到不支持这些命令的旧版后，再升级须使用已校验的新包执行 install。

更新和回退会先检查客户端及 codexgo 组件是否运行，无法检查时拒绝写入。它是操作前检查，不是后台维护锁：操作期间不要另开客户端。不会自动杀进程、断开远端或启动应用。输出的版本是磁盘上的安装版本，重新 launch 后才由新进程使用。

**下载暂存（0.1.12-preview.1，真实安装验收未完成）：** 需要等当前任务结束再替换程序时，可先下载到该安装文件旁的私有更新目录：

```sh
codexgo update --stage                # 联网校验并暂存，当前程序继续运行
codexgo update --staged               # 只读查看暂存版本、校验值和目标状态
codexgo update --apply-staged --dry-run # 只查看计划，不联网、不修复或替换
# 待任务结束、完全退出客户端及 codexgo 组件后：
codexgo update --apply-staged         # 核验原安装并应用缓存，保留回退备份
codexgo update --discard-staged       # 取消当前暂存，仅删除该暂存记录与缓存
```

这些命令都针对当前执行的 codexgo 安装路径。`--stage` 不需要退出客户端，不执行下载文件，也不修改安装目标；缓存文件权限为 0600。下载或校验失败保留原安装和已有暂存。`--staged` 和 `--discard-staged` 不查询更新源、不运行磁盘程序；应用不重新下载，会再次检查缓存 SHA／平台、计划 ID、原安装 SHA 和空闲状态，目标已被改写时拒绝覆盖。相同安装的暂存、应用、取消和直接安装共用锁；检查后有新计划出现时，旧操作会拒绝，不误处理新计划。应用期间不要另开客户端。

替换沿用现有备份与事务安装器。应用中断后再次执行 `--apply-staged`，只恢复与该暂存匹配的安装事务；若程序已替换，会完成登记和缓存清理，不再备份新程序或多安装一次。其他未完成事务须先按原流程恢复。已回退的旧暂存不能重新应用，需取消后重新检查。损坏的暂存元数据会明确拒绝；正常取消只清理该记录指向的缓存，不扫描删除目录内其他文件。极端中断可能留下无引用缓存文件，须经人工核对后清理，不影响原安装备份。

CLI 暂存默认需要显式 `--apply-staged`。符合下述条件并明确选择自动下载策略的本机安装，也可在下次 `launch` 时自动应用；远端 daemon 替换仍待接入。本地构建替换／降级时，暂存和应用两步都必须分别带对应的 `--replace-local`／`--allow-downgrade`，暂存不等于许可后续静默覆盖。

**适配器和已有会话：** 这一轮仅替换主程序。受管理 Pi／CodeBuddy 适配器仍需在插件设置更新；已有会话固定旧适配器版本，不因更新主程序自动迁移。手工 manifest、原生 CLI、账号和会话记录不修改。只有安装程序的备份可由本命令回退，不宣称同时回退配置或会话数据。

**设置面板检查更新（0.1.12-preview.1，真实安装验收未完成）：** 顶部插件设置中选择目标主机后，codexgo 区域显示该主机服务进程的运行版本；打开面板不会查询更新源。点击「检查更新」才读取固定公开下载库的正式版元数据，并显示可用版本和校验信息；不会下载程序、安装或重启。运行版本不代表磁盘文件已更新，旧进程需要安排重启后才生效。远端离线、旧服务不支持查询或元数据无法校验时会明确报错，不回退成本机结果。本地测试构建会单独标明，不能据此判断已经是最新正式版。本机自动下载与启动时应用见下文，远端安装仍未接入。

**自动检查（0.1.12-preview.1，真实安装验收未完成）：** 同一面板可开启「自动检查正式版」，默认关闭；开关保存在目标主机的 `$CODEXGO_HOME/update-policy.json`，不会改变其他主机。启用后，该主机 codexgo 服务在启动时检查一次，此后每 24 小时检查；服务退出后不再运行。每个服务进程各自计算间隔，重启会重新检查。打开设置可看最近自动检查结果；运行中的桌面每分钟读取当前主机的结果，有较新正式版时提示一次（同一页面内按主机和版本去重）。本地测试构建不与正式版比较，不弹升级提示，但设置中仍可查看正式版。

关闭开关会取消本进程自动查询并等待它退出；等待失败会明确提示刷新核对，不宣称已经完成。其他服务进程会在一分钟内重读开关。设置损坏、权限不安全或不可读取时停止自动查询；旧窗口保存时若修订号已改变，会要求刷新，避免覆盖别处的更改。自动检查失败不会保留旧成功结果充当本次结果，可用手动检查重试。默认策略为「只提示」，不会下载或安装。可选的本机自动下载策略具有下面明确限定的安装权限；服务本身不应用更新、不重启或停止会话。回退到只支持提醒的旧构建前，应先关闭自动更新并改回只提示；旧构建不识别 download 策略，会停止自动检查并提示设置不可核验。

**本机自动下载及下次启动应用（0.1.12-preview.1，真实安装验收未完成）：** 在上述面板中开启检查后，可明确选择「自动下载，完全退出后下次启动应用」。目前仅对 macOS 本机正式构建、受支持的 `codexgo` 安装位置开放；本地验收版及远端 shim 不开放此策略，仍可使用检查提醒。策略记录的是用户选定的完整安装路径，同一主机的另一份程序不能自动接管它。

发现更新后，Go 服务重新核对发布身份、版本和摘要，再下载校验并暂存，运行程序保持不变。关闭开关或改为「只提示」会撤销自动应用权限；下载提交前再次核对策略修订号和原文件摘要，其他窗口关闭后旧下载不能提交。设置面板从磁盘复核待应用版本，因此 CLI 取消暂存后不会继续显示旧计划；已有较旧暂存包与最近发现的新版分别显示。

完全退出客户端、守护进程和相关任务后，再执行同一路径的 `codexgo launch`：启动器在打开客户端前检查当前策略、暂存文件、目标版本和空闲状态，复用事务安装和备份，再重新执行新启动器。`--force` 不绕过更新的空闲检查；`--dry-run` 不下载或应用。任何失败都不会继续打开客户端，以免启动于未确认的更新状态。`version`、普通 shim/daemon 启动及打开设置都不会安装更新。回退仍使用 `codexgo rollback`；手动回退留下的旧暂存不能再次自动应用。

这次不更新，或启动前更新失败时，可直接从终端恢复，不需要先打开设置面板：

```sh
codexgo update --disable-auto  # 关闭本主机自动更新，保留程序和缓存
codexgo update --staged       # 只读核对缓存；缓存可正常读取时可继续取消
codexgo update --discard-staged
codexgo launch
```

如果策略 JSON 本身损坏，关闭命令会拒绝覆盖。先备份移走 codexgo 自有策略文件，再启动；缺少策略文件默认关闭，不会删除缓存或会话。这里仅操作 `update-policy.json`，不要移动整个数据目录：

```sh
mv "${CODEXGO_HOME:-$HOME/.codexgo}/update-policy.json" \
   "${CODEXGO_HOME:-$HOME/.codexgo}/update-policy.disabled-backup-$(date +%s).json"
codexgo launch
```

若坏暂存记录使查看/取消也失败，关闭策略后仍可启动旧程序；保留缓存供诊断，不要直接删除更新目录中的安装备份。下载状态只说明本轮下载结果，待应用版本来自当前磁盘计划；真实安装替换与客户端体验需另行验收，本批仅隔离合成安装测试。


本地验收版含 `+local.…` 标记，普通 update 会拒绝覆盖，即使发现较新正式版也须明确 `--replace-local`；较旧版本另须 `--allow-downgrade`。这些选项不会绕过包校验、运行检查或目标保护。

**内部开发的一条命令安装：** 在私有源码工作区完成审查/测试后，先退出客户端，运行：

```sh
python3 scripts/install-local.py --dry-run  # 编译并查看安装计划
python3 scripts/install-local.py            # 编译、标记本地版本并安装
```

脚本通过临时 overlay 添加本地版本标记，不修改受跟踪源码；调用与正式更新相同的 Go 安装逻辑，默认安装到已有 `~/.local/bin`，自定义目录传 `--dir`。再次内部安装允许替换之前的本地验收版，仍保留备份并拒绝意外降级。这不是发版脚本，不上传任何文件，不修改公开仓库。已下载并校验的新版程序也可执行 `/path/to/codexgo install --dir /existing/bin` 使用同一套安装逻辑。


**Mac：** 下载并校验新包，等待任务结束并退出桌面版。把旧启动文件复制为带旧版本号的备份，再用新包替换同一路径。执行 `version`，从这个路径重启。回退时退出后换回旧文件。仅想回到原版 Codex，退出后从 Dock 启动即可。

**开发版远端校验更新与回退（0.1.12-preview.1，真实安装验收未完成）：** 在目标主机执行以下命令。它们只处理既有 `$CODEXGO_HOME/bin/codex` 安装，不改 shell 配置，不下载原生 Codex 或 harness CLI，也不会替你停止 daemon：

```sh
/path/to/codexgo remote update --check
/path/to/codexgo remote update --dry-run
# 等待任务完成、断开所有客户端，再自行执行已支持的 remote stop
/path/to/codexgo remote update
/path/to/codexgo remote rollback --dry-run
/path/to/codexgo remote rollback
```

`remote update` 使用本机更新相同的固定发布源和校验逻辑；`--version` 可选正式版本，本地构建替换须 `--replace-local`，降级另须 `--allow-downgrade`。上述路径可以是已安装的远端 shim（`$CODEXGO_HOME/bin/codex remote update`）；普通 `codex update` 仍交给原生 CLI，不能用它代替此入口。命令显示的是磁盘版本，不能据此断言驻留 daemon 已更新。

更新与回退全程持有既有安装锁及 daemon 锁，daemon 存活时拒绝替换，不发信号；新 daemon 在提交期间也不能取得同一锁。开发版 daemon 会在启动原生控制服务、会话管理及后台检查前先取得唯一实例锁，并连续持有到停止清理结束；并发启动落败者直接退出，初始化失败会释放锁以便重试。程序和私有回退记录一起提交，失败同时恢复，保留上一版的回退能力。回退只恢复最近一次 `remote update` 的二进制备份，保留 shell、会话、适配器、账号和原生程序。中断事务恢复算一次操作，不会顺便再回退一个版本；完成恢复后确需再回退时应另行明确执行。

损坏备份、目标被手工修改、符号链接或不认可的恢复路径会拒绝操作并保留文件供核验。如果 pending journal 属于旧 `remote install/uninstall` 的 shell 配置事务，新入口会拒绝接管；应使用原安装操作恢复，不要删除 journal 或手动覆盖用户配置。本批未对真实主机执行更新、回退或停止。

**开发版 Linux 远端自动更新（0.1.12-preview.1，真实安装验收未完成）：** 从既有托管路径 `$CODEXGO_HOME/bin/codex` 启动的正式构建，设置面板可按所选远端选择“只提示”或“自动下载，下次 daemon 启动应用”。默认关闭；只提示不授权安装。其他路径、本地验收构建和无法读取 `/proc/self/exe` 的环境不支持自动下载。macOS 远端安装保持检查提醒，暂不自动应用。

下载缓存与程序分开，当前 daemon 和会话继续运行，不主动停止。任务结束后由你按既有流程停止 daemon，下次启动会在控制服务和会话管理初始化前，复核当前开关、安装目标、加载程序 SHA、暂存摘要及版本。仍持有启动锁时复用远端更新事务，成功后重新执行新程序，再开始初始化。加载旧程序的竞争启动者也必须复核并重新执行已验证的新版本；重新执行失败不会继续启动旧服务。

在**目标远端主机**可以使用以下恢复入口，无需执行磁盘上的旧程序来查询版本，也不下载新包：

```sh
/path/to/codexgo remote update --disable-auto
/path/to/codexgo remote update --staged
/path/to/codexgo remote update --discard-staged
```

关闭开关或改为只提示会阻止后续暂存提交和自动应用；已经完成替换的文件不会因此回退。取消缓存不会自动关闭策略，如要阻止再次下载，请先关闭自动更新。回退后旧暂存计划不会被自动重装，用户明确取消该计划后才允许重新下载。损坏的普通缓存可取消，无法核验的元数据或链接会保留供诊断；策略损坏时先备份并移走 codexgo 自有 `update-policy.json` 后重试，缺失策略默认关闭。

未完成的安装事务**不会**在自动启动中自行恢复：启动会报错退出，应先关闭自动更新，使用 `remote rollback` 显式恢复本更新器事务，再检查/取消旧计划；旧 shell 安装事务仍由原安装器处理。回退记录和用户数据保持不变。这里的验证以隔离目录、合成程序和 Linux CI 为依据，不等于真实 SSH 主机或用户会话已验收。

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

### Claude 原生模型目录（0.1.12-preview.2）

模型列表从当前 Claude CLI 控制通道读取具体模型和 Effort 档位，选择时保留模型 ID 与上下文后缀；不通过聊天请求探测。刷新失败保留旧目录并标为待刷新，可在插件设置重试。显式 `claude_models` 配置（包括空列表）仍优先于自动目录。活动会话继续使用启动时的能力快照，新目录用于新会话。


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
