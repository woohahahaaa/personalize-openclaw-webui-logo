# Personalize OpenClaw WebUI Logo

Personalize the OpenClaw WebUI (control panel) favicon — i.e. the OpenClaw /
crawfish browser-tab logo. Deploys a bundled logo set
(`logos/<set-name>/`) to all favicon slots in the control-ui dist, or updates
the logo library from a hub.

Optional **`gateway:startup` auto-restore hook** re-applies your logo
whenever an `openclaw` upgrade overwrites `dist/`.

Cross-platform: macOS / Linux / Windows (Git Bash or WSL).

## Two SKILL.md variants

This repo ships two language variants of the same skill, mirroring what's
published on each hub. The code (scripts + `logos/` + `hooks/`) is shared;
only the SKILL.md describing usage is in two languages.

| Path | Language | Mirrors |
|------|----------|---------|
| [`clawhub/SKILL.md`](./clawhub/SKILL.md) | English | [ClawHub](https://clawhub.ai/) (`clawhub install personalize-openclaw-webui-logo`) |
| [`skillhub/SKILL.md`](./skillhub/SKILL.md) | 中文 | [SkillHub](https://skill.xfyun.cn/) (`skillhub install personalize-openclaw-webui-logo`) |

## Repo layout

```
personalize-openclaw-webui-logo/
├── README.md
├── LICENSE
├── skillhub/SKILL.md            # 中文 SKILL.md (SkillHub mirror)
├── clawhub/SKILL.md             # English SKILL.md (ClawHub mirror)
├── apply-logo.sh                # generic apply script
├── check-and-restore.sh         # gateway:startup hook helper (hash check + re-apply)
├── install-hook.sh              # register the auto-restore hook
├── uninstall-hook.sh            # unregister the auto-restore hook
├── hooks/logo-guard/            # workspace-hook source (HOOK.md + handler.js)
└── logos/<set-name>/            # one master per set (base64-encoded PNG)
```

## Install

### Option 1: From SkillHub (中文使用者推荐)

```bash
skillhub install personalize-openclaw-webui-logo
```

### Option 2: From ClawHub (English users)

```bash
clawhub install personalize-openclaw-webui-logo
```

### Option 3: Manual

```bash
# Clone (via mainland-China mirror if direct GitHub is slow)
git clone https://gh-proxy.com/https://github.com/woohahahaaa/personalize-openclaw-webui-logo.git

# Apply a logo set
cd personalize-openclaw-webui-logo
bash apply-logo.sh mac        # or: linux, windows, mac-finder, original
```

Then hard-refresh the OpenClaw WebUI (Cmd+Shift+R / Ctrl+F5).

## License

MIT — see [LICENSE](./LICENSE).
