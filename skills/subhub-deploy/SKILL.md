---
name: subhub-deploy
description: "Kejizero订阅转换（SubHub）单镜像部署：docker-compose/docker 双模式、subconverter 源码编译、Caddy 反代、短链服务、GitHub Actions 镜像构建"
---

# SubHub 订阅转换全家桶部署

仓库: https://github.com/MinimaxFlora/Subhub
镜像: `zhaoweiwen123/subhub:latest`（Docker Hub，GitHub Actions 手动触发构建）
前端基础: [cmliu/sub-web-modify](https://github.com/cmliu/sub-web-modify)（Vue2 + Element UI，品牌改为 Kejizero订阅转换）

## 架构（单镜像，一个容器跑全部）

```
SubHub 镜像
├── Caddy          # 网关：伺服前端静态文件 + 反代后端/短链 + 自动 HTTPS
├── subconverter   # 后端（内置源码本地编译，v0.9.9）
├── shortlink      # 短链服务（Python + SQLite，myurls 协议）
└── supervisord    # 管理上面三个进程
```

端口: 8080(前端) / 25500(后端) / 7999(短链) / 80+443(域名模式)

## 双模式部署

### 本机模式（不填域名 → IP+端口访问）
```bash
docker compose up -d
# 前端 http://IP:8080 | 后端 :25500 | 短链 :7999
```

### 域名模式（填三个域名 → Caddy 反代 + HTTPS）
```ini
# .env（与 docker-compose.yml 同目录）
FRONTEND_DOMAIN=sub.example.com
BACKEND_DOMAIN=api.example.com
SHORTLINK_DOMAIN=short.example.com
BACKEND_URL=https://api.example.com
SHORTLINK_URL=https://short.example.com
ACME_EMAIL=admin@example.com
```
```bash
docker compose up -d
# https://sub.example.com | api. | short.
```

### 纯 docker run
```bash
docker run -d --name kejizero-subhub --restart=always \
  -p 8080:8080 -p 25500:25500 -p 7999:7999 \
  -v subhub_data:/data zhaoweiwen123/subhub:latest
# 域名模式加 -p 80:80 -p 443:443 和 -e 三个域名
```

## 关键踩坑记录（务必记住）

1. **Docker Hub 镜像名必须全小写**：`SubHub` 会报 `repository name must be lowercase`，用 `subhub`
2. **Alpine supervisor 包名是 `supervisor`**，不是 `py3-supervisor`（后者 404 不存在）
3. **sass 最新版要求 Node≥20.19**：前端构建阶段用 `node:22-alpine`，不能用 node:18
4. **subconverter 配置路径坑**：源码 `setcd(argv[0])` 会切到程序目录（/usr/bin）找 pref.ini → 找不到就用默认配置且只监听 127.0.0.1！**必须显式指定**：`subconverter -f /base/pref.ini`（supervisord.conf 里配）
5. **前端默认后端/短链是编译期写死的**（VUE_APP_*）：必须构建镜像时注入 `BACKEND_URL`/`SHORTLINK_URL`，运行时 environment 改不了。GitHub Actions 从 `.env.example` 读取域名 → **改域名要同时改 `.env.example` 并重新构建镜像**
6. **docker compose 多文件合并 ports 是追加**：本机已有 Caddy 占用 80/443 时，用 `docker-compose.local.yml` + `ports: !override` 强制替换
7. **短链服务必须支持 multipart/form-data**：前端用 FormData 提交（不是 urlencoded），用 `cgi.FieldStorage` 解析，否则报 "longUrl 为空"
8. **系统 Caddy 占用 80/443 时**：容器以本机模式跑，由系统 Caddy 反代域名（效果一致）；干净服务器上镜像内 Caddy 全接管

## 镜像构建（GitHub Actions）

- 触发：Actions → Build & Push Docker Image → Run workflow（手动，不自动）
- 流程：读 `.env.example` 域名 → build-args 注入前端 → buildx 多阶段构建 → push `zhaoweiwen123/subhub:latest`
- 多阶段：node:22 构建前端 → alpine 编译 subconverter（5-15分钟）→ 合成运行镜像
- 所需 Secrets：`DOCKERHUB_USERNAME`、`DOCKERHUB_TOKEN`

更新流程：
```bash
docker pull zhaoweiwen123/subhub:latest
docker compose up -d --no-build   # 或 docker compose -f docker-compose.yml -f docker-compose.local.yml up -d
```

## 短链服务（shortlink/server.py）

- 接口：`POST /short`，form 字段 `longUrl`（base64 编码 URL）+ 可选 `shortKey`
- 返回：`{"Code":1,"ShortUrl":"https://short.域名/xxx"}` 或 `{"Code":0,"Message":"..."}`
- 同 URL 复用短码；自定义短码 1-32 位字母数字下划线横线
- 数据：SQLite `/data/urls.db`（volume 持久化）
- 环境变量：`BASE_URL`（生成短链的展示域名）、`DB_PATH`、`HTTP_PORT`

## 常用排障

```bash
docker logs -f kejizero-subhub
docker exec kejizero-subhub netstat -tlnp   # 检查 8080/25500/7999 监听
docker exec kejizero-subhub sh -c 'grep -ro "api\.kejizero\.xyz\|localhost:25500" /app/web/js/*.js'  # 检查前端注入地址
curl https://api.域名/version   # 后端健康检查，应返回 subconverter vX.X.X backend
```

## 相关

- 系统 Caddy 配置：`/etc/caddy/Caddyfile`（本机反代三个域名 + openclaw）
- 本机部署：`docker-compose.local.yml` 覆盖端口映射（!override 去掉 80/443）
- 前端品牌修改：`frontend/public/logo.png`、`frontend/src/views/Subconverter.vue`
