# My-Skills 🦞

小龙虾的个人 OpenClaw 技能仓库 —— 存放 OpenWrt 固件构建生态与订阅转换部署相关的可复用技能。

## 技能列表

| Skill | 用途 |
|---|---|
| [openwrt-imagebuilder-action](skills/openwrt-imagebuilder-action/SKILL.md) | gh-action-imagebuilder 固件构建 action：机制、inputs、OpenClash 内核内置、tag/release 维护 |
| [extras-paclages-repo](skills/extras-paclages-repo/SKILL.md) | Extras_Paclages 插件库：三分支结构、签名索引、gen-index.sh、加包流程 |
| [subhub-deploy](skills/subhub-deploy/SKILL.md) | Kejizero订阅转换（SubHub）单镜像部署：docker-compose/docker 双模式、subconverter 源码编译、Caddy 反代、短链服务、GitHub Actions 镜像构建 |

## 使用方式

每个技能是标准 OpenClaw skill 结构：

```text
skills/<skill-name>/
  SKILL.md    # YAML frontmatter（name + description）+ 正文
```

本地使用：clone 仓库后，把 `skills/` 下的目录软链或复制到 `~/.openclaw/skills/`（或 OpenClaw 配置的 skills 目录）即可被自动加载。

## 维护

新增技能：

1. 在 `skills/<skill-name>/SKILL.md` 写标准格式（frontmatter 必须有 `name` + `description`）
2. 验证：`python3 -c "import yaml; yaml.safe_load(open('skills/<name>/SKILL.md').read().split('---',2)[1])"`
3. 提交推送（身份 `MinimaxFlora <zj18139624826@gmail.com>`）

## 相关仓库

- [gh-action-imagebuilder](https://github.com/MinimaxFlora/gh-action-imagebuilder) — 固件构建 action
- [Extras_Paclages](https://github.com/MinimaxFlora/Extras_Paclages) — 插件库
- [Firmware-Build](https://github.com/MinimaxFlora/Firmware-Build) — 一键构建 workflow
- [Subhub](https://github.com/MinimaxFlora/Subhub) — Kejizero订阅转换全家桶（单镜像 + Caddy + subconverter + 短链）
