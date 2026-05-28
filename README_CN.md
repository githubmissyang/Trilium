# Trilium Notes 中文增强版

基于 [TriliumNext/Trilium](https://github.com/TriliumNext/Trilium) 上游仓库，增加以下增强：

- **简体中文翻译 100% 覆盖** — 客户端和服务端翻译均已补齐
- **1panel 应用商店模板** — 支持通过 1panel 一键部署
- **自定义镜像构建** — GitHub Actions 自动构建多架构 Docker 镜像，推送到 `docker.1ms.run`
- **内置 TOTP 双因子认证** — 可在设置中开启，保护笔记安全

---

## 与上游的区别

| 内容 | 上游仓库 | 本 Fork |
|------|----------|---------|
| 简体中文翻译（客户端） | ~94% | **100%** |
| 简体中文翻译（服务端） | ~97% | **100%** |
| 1panel 应用模板 | 无 | 有 |
| 自定义镜像 CI/CD | 仅推送到 GHCR/DockerHub | 额外推送到 `docker.1ms.run` |
| 部署文档 | 无 1panel 指南 | 有详细中文部署文档 |

所有改动仅涉及翻译文件、配置文件和文档，**不修改任何核心源码**，可无缝跟随上游更新。

---

## 快速开始

### Docker 部署（推荐）

```bash
docker run -d \
  --name trilium \
  -p 8080:8080 \
  -v ~/trilium-data:/home/node/trilium-data \
  docker.1ms.run/githubmissyang/trilium:latest
```

访问 `http://localhost:8080`，首次使用设置密码即可。

### 1panel 一键部署

详见 [1panel 部署指南](./docs/1panel-deployment.md)

1. 将 `1panel/apps/trilium/` 复制到服务器 `/opt/1panel/resource/apps/local/trilium/`
2. 在 1panel 应用商店 → 本地中安装
3. 配置端口和时区，一键启动

---

## 开启双因子认证

Trilium 内置 TOTP 双因子认证，强烈建议开启：

1. 登录后进入 **设置 → 多因子认证**
2. 开启 MFA，选择 TOTP 方式
3. 点击 **生成密钥**，用 Google Authenticator / Microsoft Authenticator 扫码
4. 点击 **生成恢复码**，妥善保存

开启后登录页会要求输入 6 位验证码。

---

## 跟随上游更新

```bash
# 添加上游仓库（首次）
git remote add upstream https://github.com/TriliumNext/Trilium.git

# 同步上游代码
git fetch upstream
git merge upstream/main

# 检查新增的翻译 key
python3 -c "
import json
with open('apps/client/src/translations/en/translation.json') as f: en = json.load(f)
with open('apps/client/src/translations/cn/translation.json') as f: cn = json.load(f)
missing = [k for k in en if k not in cn]
print(f'缺失 {len(missing)} 个翻译 key')
for k in sorted(missing): print(f'  {k}')
"

# 补齐翻译后，构建新镜像
git tag v0.91.0
git push origin v0.91.0
```

---

## 镜像构建

### 配置 GitHub Secrets

在仓库 Settings → Secrets and variables → Actions 中添加：

| Secret | 说明 | 示例 |
|--------|------|------|
| `REGISTRY_URL` | Registry 地址 | `docker.1ms.run` |
| `REGISTRY_USERNAME` | 用户名 | - |
| `REGISTRY_PASSWORD` | 密码/Token | - |
| `IMAGE_NAME` | 镜像名 | `githubmissyang/trilium` |

### 触发构建

- 推送 tag（如 `v0.90.3`）自动触发
- 或在 GitHub Actions 页面手动触发

构建产物：`docker.1ms.run/githubmissyang/trilium:<版本>` + `latest`

支持架构：`linux/amd64`、`linux/arm64`

---

## 环境变量

### 基础配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_DATA_DIR` | 数据存储路径 | `/home/node/trilium-data` |
| `TRILIUM_NETWORK_PORT` | 服务端口 | `8080` |
| `TRILIUM_SESSION_COOKIEMAXAGE` | 会话有效期（秒） | `1814400`（21天） |

### 安全配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_GENERAL_NOAUTHENTICATION` | 禁用认证（不推荐） | `false` |
| `TRILIUM_NETWORK_TRUSTEDREVERSEPROXY` | 信任反向代理头 | `false` |

### OAuth/SSO

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_OAUTH_BASE_URL` | OAuth 回调地址 | 空 |
| `TRILIUM_OAUTH_CLIENT_ID` | 客户端 ID | 空 |
| `TRILIUM_OAUTH_CLIENT_SECRET` | 客户端密钥 | 空 |
| `TRILIUM_OAUTH_ISSUER_BASE_URL` | 发行方地址 | `https://accounts.google.com` |

---

## 反向代理配置

Nginx 示例：

```nginx
server {
    listen 443 ssl;
    server_name trilium.example.com;

    ssl_certificate     /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

需设置环境变量 `TRILIUM_NETWORK_TRUSTEDREVERSEPROXY=true`。

---

## 本 Fork 的文件结构

```
（新增/修改的文件）
├── .github/workflows/build-custom-image.yml   # CI/CD 工作流
├── 1panel/apps/trilium/                       # 1panel 应用模板
│   ├── logo.png
│   ├── README.md
│   ├── data.yml                               # 应用元数据
│   └── <版本>/docker-compose.yml              # Compose 配置
├── docs/1panel-deployment.md                  # 部署文档
├── apps/client/src/translations/cn/translation.json  # 客户端中文翻译（已补齐）
└── apps/server/src/assets/translations/cn/server.json # 服务端中文翻译（已补齐）
```

---

## 致谢

- [TriliumNext/Trilium](https://github.com/TriliumNext/Trilium) — 上游项目
- [Nriver/trilium-translation](https://github.com/Nriver/trilium-translation) — 早期中文翻译项目

---

## 许可证

与上游一致，采用 [AGPL-3.0](./LICENSE) 许可证。
