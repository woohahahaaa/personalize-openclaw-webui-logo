---
name: personalize-openclaw-webui-logo
slug: personalize-openclaw-webui-logo
displayName: 个性化 OpenClaw 网页 Logo
version: 2.4.2
description: "个性化 OpenClaw 网页（控制台）的 favicon——即 OpenClaw / 小龙虾 网页 logo。把内置的某套 logo（logos/<套名>/）部署到 control-ui dist 的全部尺寸，或从 hub 更新 logo 库。**可选 `gateway:startup` 自动还原钩子**：openclaw 升级覆盖 dist 后自动重贴你的 logo。运行时状态（已应用时间戳 / 日志）写在 `~/.openclaw/state/` 而非 skill 目录内，所以 skill 发布到 hub 时不会泄漏本机路径或用户名。跨平台：macOS / Linux / Windows（Git Bash 或 WSL）。必须明确提到 openclaw 或 小龙虾 才触发。 | 触发词（须含 openclaw 或 小龙虾）：修改openclaw网页logo、换openclaw logo、换openclaw favicon、换小龙虾logo、换成XX的openclaw logo、更新openclaw logo库、装openclaw logo守护钩子、auto-restore openclaw logo、change openclaw logo、update openclaw logo library。"
metadata:
  homepage: https://github.com/openclaw/openclaw
  license: MIT
  allowed-tools: "read,write,edit,exec,image"
---

# 个性化 OpenClaw 网页 Logo（favicon / 小龙虾 logo）

> **触发词（最高优先级）—— 必须提到 openclaw 或 小龙虾 才触发：**
> 修改openclaw网页logo、换openclaw logo、换openclaw网站logo、换openclaw favicon、换小龙虾logo、换成XX的openclaw logo、还原openclaw logo、还原openclaw favicon、更新openclaw logo库、更新openclaw favicon库、**自动还原openclaw logo**、**装openclaw logo守护钩子**。
> 仅泛泛说"换 logo / 换图标"而**未提 openclaw 或 小龙虾** 时 → 不要触发本 skill。
> 看到 / 听到符合条件的触发词 → 立即按下方工作流执行。

更换 OpenClaw 控制台网站的 favicon（浏览器标签图标）。支持**多套 logo**，每套放在 `logos/<套名>/favicon-master.png`，用 `apply-logo.sh <套名>` 切换。所有 master 备份都在本 skill 里——openclaw 版本更新覆盖 dist 后一条命令还原；或者装上**可选 `gateway:startup` 自动还原钩子**，让 OpenClaw 每次重启时自己把 favicon 修回来。

## 多套机制

```
skills/personalize-openclaw-webui-logo/
├── apply-logo.sh                       # 通用脚本：bash apply-logo.sh <套名>（默认 mac）
├── check-and-restore.sh                # v2.4：hash 比对，drift 时重贴
├── install-hook.sh / uninstall-hook.sh # v2.4：注册/卸载 gateway:startup 钩子
├── hooks/logo-guard/                   # v2.4：钩子源码（HOOK.md + handler.js）
└── logos/
    └── <套名>/favicon-master.png       # 每套一个 master PNG
```

所有运行时状态（已应用时间戳、钩子日志）都放在 skill **目录之外** `${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/`，所以发布到 skillhub / clawhub 时 skill 本体永远干净，不会泄漏本机绝对路径或用户名。

- 现有套：`original`、`mac`、`windows`、`linux`、`mac-finder`。
- 列出所有套：`ls skills/personalize-openclaw-webui-logo/logos/`

## 场景

### A. 换成某套现有 logo（"换成 XX 的"）
1. 跑 `bash <skill_dir>/apply-logo.sh <套名>`（如 `apply-logo.sh mac`）。
2. 脚本自动定位 control-ui dist、自动选可用的图像工具，然后从该套 master 生成全部尺寸写入 dist，**并在 `~/.openclaw/state/personalize-openclaw-webui-logo/applied-state` 写下时间戳**，供自动还原钩子（场景 C）使用。
3. 提示用户 Cmd+Shift+R / Ctrl+F5 硬刷新。
> 套名不存在时脚本会列出所有可用套并报错。

### B. 更新 logo 库（"更新 openclaw logo 库 / 更新 openclaw favicon 库"）
从 hub 拉本 skill 的最新版（包含最新 logo 套）。已发布到 **clawhub** + **skillhub**（slug: `personalize-openclaw-webui-logo`）。
1. 优先 `clawhub update personalize-openclaw-webui-logo`（或 `clawhub install ...`）。
2. 也可 `skillhub upgrade personalize-openclaw-webui-logo`。
3. 升级后 `ls skills/personalize-openclaw-webui-logo/logos/` 确认新套到位。
4. 如需立即生效，跑 `apply-logo.sh <套名>` 重新部署。
> logo master 以 base64 `.txt` 形式随包携带（部分 hub 不收 `.png`）；apply-logo.sh 会自动解码。

### C. 装自动还原钩子（"自动还原 openclaw logo" / "装 openclaw logo 守护钩子"）

OpenClaw 自升级（比如 `npm update openclaw`）时 `dist/control-ui/` 整个会被官方版覆盖。与其每次升完手动跑 `apply-logo.sh`，不如装一个 **`gateway:startup` 内部钩子**：每次 gateway 启动时算一次哈希，如果 favicon 漂移了就自动重贴回上次用的那套。

**为什么便宜。** 钩子靠事件驱动，每次 gateway 启动只触发一次（不是轮询）。触发时：
1. 读 `~/.openclaw/state/personalize-openclaw-webui-logo/applied-state`（`apply-logo.sh` 写的时间戳）。
2. 算 `sha256(dist/control-ui/favicon-32.png)` —— 单个文件，~32 KB，几毫秒。
3. hash 跟时间戳一致 → 直接退出。零工作量。
4. hash 对不上 → 后台（非阻塞）跑 `check-and-restore.sh`，它再调 `apply-logo.sh <套名>` 重贴 + 更新时间戳。

钩子**不阻塞** gateway 启动 —— 重活全在 detached 子进程里干。

**安装：**
```bash
bash <skill_dir>/install-hook.sh
```

这个脚本会：
1. 检查 `~/.openclaw/state/personalize-openclaw-webui-logo/applied-state` 时间戳存在（否则没东西可守 —— 先跑 `apply-logo.sh <套名>`）。
2. 把 `<skill_dir>/hooks/logo-guard` **拷贝**到 `~/.openclaw/hooks/logo-guard`（managed hooks 目录）。不用 symlink，因为 OpenClaw 的 hook discovery 不跟随 symlink。
3. 在拷贝里写一个 `.skill-dir` 指针，让 handler 能找回真正的 skill 目录（即使你之后挪过 skill 位置）。
4. Managed hooks 默认就是启用状态，不需要再跑 `openclaw hooks enable`。
5. 钩子**需要下一次 gateway restart 后才生效** —— 此脚本不会替你 restart。

**验证：**
```bash
openclaw hooks list | grep logo-guard                                          # 应该看到 enabled
openclaw hooks info logo-guard                                                 # 完整信息
tail -f "${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/logo-guard.log"  # 看最近一次启动钩子的输出
```

**测试。** 把 favicon 砸坏再 restart：
```bash
echo "broken" > "$(npm root -g)/openclaw/dist/control-ui/favicon-32.png"
# 你手动 restart gateway
# restart 后日志里应该看到 DRIFT detected → re-apply OK
```

**卸载：**
```bash
bash <skill_dir>/uninstall-hook.sh
```

会移除 `~/.openclaw/hooks/logo-guard`。dist 里的 favicon 文件保持原状。

### D. 刚装完欢迎提示（v2.4.2 新增）——问一次，不重复问

当 agent 检测到本 skill **刚装上且从未贴过** 时触发。以下**任一**都算“新装”信号：

- agent 本轮刚跑过 `skillhub install personalize-openclaw-webui-logo` 或 `clawhub install personalize-openclaw-webui-logo`。
- 用户主动说了类似“我刚装了 openclaw logo 这个 skill / I just installed the openclaw logo skill”。
- 状态文件 `${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/applied-state` **不存在**（`test -f "$HOME/.openclaw/state/personalize-openclaw-webui-logo/applied-state"` 返回非零）。

命中任一个信号 → 用当前渠道的自然语言，向用户问**一次**类似：

> “看起来你刚装好 OpenClaw logo skill ✨ 要不要现在就换个 favicon？我可以随机帮你选一套，不想选也可以。”

然后：

- **用户说 好 / 随便 / 随机 / 帮我选 / yes / random** → 列 `<skill_dir>/logos/` 下除 `original` 以外的所有套（`original` 是 OpenClaw 默认 favicon，“随机到原版”没意思），均匀随机抽一套，跑 `bash <skill_dir>/apply-logo.sh <选中的套名>`，告诉用户选的哪套，提醒 Cmd+Shift+R / Ctrl+F5 硬刷新。
- **用户明确指定某套** → 直接 `apply-logo.sh <那套>`。
- **用户说 不用 / 算了 / 晚点再说 / no** → 说声好的、停。**以后不要再问**；用户想换时会主动说“换 openclaw logo”。
- **用户说好并且顺便要装自动还原钩子** → 贴完后加跑 `bash <skill_dir>/install-hook.sh`（场景 C）。

一旦 `applied-state` 存在就不再提示了 —— skill 不再是“新装”状态。之后重装（比如 `skillhub install --force`）也不会重新触发，这是故意的：时间戳在 skill 目录**之外**，跨重装保留。

## 关键路径

| 用途 | 路径 |
|------|------|
| 通用脚本 | `<skill_dir>/apply-logo.sh` |
| 各套 master | `<skill_dir>/logos/<套名>/favicon-master.png`（或 `.txt`） |
| 已应用时间戳 | `${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/applied-state`（套名 + favicon-32 哈希） |
| 自动还原核心 | `<skill_dir>/check-and-restore.sh` |
| 钩子源码 | `<skill_dir>/hooks/logo-guard/{HOOK.md,handler.js}`（源头；install-hook.sh 会拷贝到 `~/.openclaw/hooks/`） |
| 钩子日志 | `${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/logo-guard.log` |
| 生效目标 dist | **自动探测**（见下） |
| dist 内 4 个文件 | `favicon.svg` / `favicon-32.png` / `favicon.ico` / `apple-touch-icon.png` |
| index.html 引用 | `<link rel="icon" href="./favicon.svg">` 等（SVG 优先） |

### dist 自动探测顺序

1. `$OPENCLAW_UI_DIR` 环境变量（最高优先，绝对路径）。
2. `$(npm root -g)/openclaw/dist/control-ui` —— 通用，覆盖 npm/pnpm/yarn 全局安装。
3. `which openclaw` 的实际路径上溯 `../lib/node_modules/openclaw/dist/control-ui`。
4. 常见前缀：`~/.npm-global`、`~/.local`、`/usr/local`、`/opt/homebrew`、`/usr/lib`、Windows `%APPDATA%/npm`、`$USERPROFILE/AppData/Roaming/npm`。

都没命中 → 报错并提示设置 `OPENCLAW_UI_DIR=/abs/path/to/openclaw/dist/control-ui` 重试。

## 跨平台图像工具链

脚本自动选第一个可用工具（按优先级）：

| 工具 | 平台 | 安装 |
|------|------|------|
| `sips` | macOS 自带 | 无需安装 |
| `magick` / `convert` (ImageMagick) | macOS / Linux / Windows | `brew install imagemagick` / `apt install imagemagick` / `choco install imagemagick` |
| `python3 + Pillow` | 全平台 | `pip install Pillow` |
| `ffmpeg` | 全平台 | `brew/apt/choco install ffmpeg`（兜底，不生成真 `.ico`） |

都没有 → 脚本会报错并打印每个平台的安装命令。

### Windows 注意

- 用 **Git Bash**、**WSL**、或 **MSYS2** 跑 `bash apply-logo.sh`；原生 cmd/PowerShell 不支持。
- 路径自动探测覆盖 `%APPDATA%/npm/node_modules/openclaw/dist/control-ui` 和 `$USERPROFILE/AppData/Roaming/npm/...`。
- 硬刷新快捷键：Ctrl+F5。

## 注意事项

- dist 在 openclaw 包目录内，**每次 `npm update` / 版本升级会被覆盖** → master 留在 skill 里就是为此，v2.4 的自动还原钩子也是为此。
- 浏览器对 favicon 缓存很顽固，换完务必硬刷新（Cmd+Shift+R / Ctrl+F5）或清站点数据。
- 复杂图缩到 16–32px 会糊成色块，属正常；需要清爽小图标可手动简化后替换该套 master。
- 自动还原钩子只守 `apply-logo.sh` 最后一次写入的东西。如果你绕过 `apply-logo.sh` 手动塞文件进 dist，时间戳就过期了，钩子会按 stamp 记录的套把它重置掉。

## Changelog

- **2.4.2** — 纯文档增量：新增场景 D（刚装完欢迎提示）。agent 检测到本 skill 刚装上、从未贴过（`applied-state` 状态文件不存在）时，主动问用户一次是否现在要贴一套 —— 可以从 `logos/` （排除 `original`）随机抽一套。只问一次，不重复问。不改代码，完全复用现有的 `apply-logo.sh <套名>`流程。
- **2.4.1** — 把运行时状态（`applied-state` 时间戳 + `logo-guard.log`）从 skill 目录搬到 `${OPENCLAW_HOME:-$HOME/.openclaw}/state/personalize-openclaw-webui-logo/`。skill 目录自此发布安全（hub bundle 里不再夹带本机路径或用户名）。同时把 Windows `%APPDATA%/npm` 和 `$USERPROFILE/AppData/Roaming/npm` 的 dist 探测路径补全到 `check-and-restore.sh`。**迁移**：如果你装的是 v2.4.0，运行一次 `apply-logo.sh <套名>` 把时间戳写到新位置即可；老的 `<skill>/.applied-state` 和 `<skill>/.logo-guard.log` 可直接删。
- **2.4.0** — 新增：可选 `gateway:startup` 自动还原钩子（`hooks/logo-guard/`）。`apply-logo.sh` 会写一份时间戳。新增 `check-and-restore.sh`、`install-hook.sh`、`uninstall-hook.sh`。
- **2.3.0** — 移除硬编码用户路径；新增 dist 自动探测（npm root / which / 常见前缀 / `OPENCLAW_UI_DIR` 覆盖）；图像工具从 sips-only 改为 sips → ImageMagick → Pillow → ffmpeg 自动 fallback；新增 Linux/Windows 说明。
- **2.2.0** — 多套机制（logos/<套名>/），脚本一键切换。
