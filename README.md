# My-Skills 🦞

MinimaxFlora 的个人 Hermes Agent 技能仓库 —— 存放实战中沉淀的可复用技能：固件构建、订阅转换、服务部署等，持续更新。

## 技能列表

| Skill | 用途 |
|---|---|
| [subhub-deploy](skills/subhub-deploy/SKILL.md) | Kejizero订阅转换（SubHub）单镜像部署：docker-compose/docker 双模式、subconverter 源码编译、Caddy 反代、短链服务、GitHub Actions 镜像构建 |
| [openwrt-imagebuilder-action](skills/openwrt-imagebuilder-action/SKILL.md) | gh-action-imagebuilder 固件构建 action：机制、inputs、OpenClash 内核内置、tag/release 维护 |
| [extras-paclages-repo](skills/extras-paclages-repo/SKILL.md) | Extras_Paclages 插件库：三分支结构、签名索引、gen-index.sh、加包流程 |

## 使用方式

每个技能是标准 skill 结构：

```text
skills/<skill-name>/
  SKILL.md    # YAML frontmatter（name + description）+ 正文
```

### 在 Hermes 中使用

clone 仓库后，把 `skills/` 下的目录复制到 Hermes 的 skills 目录即可被自动加载：

```bash
git clone https://github.com/MinimaxFlora/My-Skills.git
mkdir -p ~/.hermes/skills
cp -r My-Skills/skills/* ~/.hermes/skills/
```

> 提示：Hermes 对 `description` 有 60 字符的系统提示预算限制（超出会被截断）。若本地加载报错，请将 description 压缩到 60 字符以内，详细说明移入正文。仓库内保留完整版描述。

### 在其他平台使用

同样适用于 Claude/OpenClaw 等支持标准 skill 结构的工具：将技能目录复制或软链到对应平台的 skills 目录即可。

## 维护

新增技能：

1. 在 `skills/<skill-name>/SKILL.md` 写标准格式（frontmatter 必须有 `name` + `description`）
2. 验证：`python3 -c "import yaml; yaml.safe_load(open('skills/<name>/SKILL.md').read().split('---',2)[1])"`
3. 提交推送（身份 `MinimaxFlora <zj18139624826@gmail.com>`）

## 相关仓库

- [Subhub](https://github.com/MinimaxFlora/Subhub) — Kejizero订阅转换全家桶（单镜像 + Caddy + subconverter + 短链）
- [gh-action-imagebuilder](https://github.com/MinimaxFlora/gh-action-imagebuilder) — 固件构建 action
- [Extras_Paclages](https://github.com/MinimaxFlora/Extras_Paclages) — 插件库
- [Firmware-Build](https://github.com/MinimaxFlora/Firmware-Build) — 一键构建 workflow
