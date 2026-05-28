# Trilium Notes 1panel 部署指南

本文档介绍如何通过 1panel 应用商店一键部署 Trilium Notes，并开启双因子认证保护笔记安全。

## 前置条件

- 已安装 1panel 面板
- 拥有 Docker Registry 账号（默认使用 `docker.1ms.run`）
- 已 Fork 本仓库到自己的 GitHub 账号

---

## 一、初始设置（一次性）

### 1. Fork 仓库

在 GitHub 上 Fork [TriliumNext/Trilium](https://github.com/TriliumNext/Trilium) 到你的账号。

本仓库已包含：
- 完整的简体中文翻译（客户端 100%、服务端 100%）
- 1panel 应用模板
- 自定义镜像构建工作流

### 2. 配置 GitHub Secrets

在 Fork 的仓库中，进入 **Settings → Secrets and variables → Actions**，添加以下 Secrets：

| Secret | 说明 | 示例值 |
|--------|------|--------|
| `REGISTRY_URL` | Docker Registry 地址 | `docker.1ms.run` |
| `REGISTRY_USERNAME` | Registry 用户名 | `your-username` |
| `REGISTRY_PASSWORD` | Registry 密码或 Access Token | - |
| `IMAGE_NAME` | 镜像名称（含命名空间） | `githubmissyang/trilium` |

### 3. 修改 1panel 模板中的镜像地址

编辑 `1panel/apps/trilium/<版本>/docker-compose.yml`，确认镜像地址正确：

```yaml
image: docker.1ms.run/githubmissyang/trilium:v0.90.3
```

---

## 二、构建 Docker 镜像

### 触发构建

**方式一：推送 Tag**

```bash
git tag v0.90.3
git push origin v0.90.3
```

**方式二：手动触发**

进入 GitHub Actions 页面，选择 **Build Custom Docker Image** 工作流，点击 **Run workflow**，输入版本号。

构建流程：`pnpm install → server:build → docker buildx（amd64 + arm64）→ 推送到 docker.1ms.run`

### 验证构建

构建完成后，镜像地址为：

```
docker.1ms.run/githubmissyang/trilium:<版本号>
docker.1ms.run/githubmissyang/trilium:latest
```

---

## 三、1panel 安装 Trilium

### 1. 安装应用模板

将项目中的 `1panel/apps/trilium/` 目录复制到 1panel 服务器：

```bash
scp -r 1panel/apps/trilium/ root@<服务器IP>:/opt/1panel/resource/apps/local/trilium/
```

### 2. 在 1panel 中安装

1. 登录 1panel 面板
2. 进入 **应用商店 → 本地**
3. 找到 **Trilium Notes**，点击安装
4. 配置端口和时区
5. 点击确认安装

### 3. 访问 Trilium

安装完成后，访问 `http://<服务器IP>:<配置的端口>`，首次使用需设置登录密码。

---

## 四、开启双因子认证（2FA）

Trilium 内置 TOTP 双因子认证，强烈建议开启。

### 步骤

1. 登录 Trilium
2. 点击左上角菜单 → **Options**（选项）
3. 选择 **Multi Factor Authentication**（多因子认证）
4. 勾选 **Enable MFA**（启用多因子认证）
5. 选择 **TOTP** 方式
6. 点击 **Generate Secret**（生成密钥）
7. 复制弹窗中显示的密钥，使用以下任一应用扫码或手动输入：
   - Google Authenticator
   - Microsoft Authenticator
   - Authy
   - 1Password
8. 点击 **Generate Recovery Keys**（生成恢复码）
9. **务必妥善保存恢复码**，丢失认证器时可用恢复码登录

### 验证 2FA

退出登录后重新访问，登录页会出现 TOTP Token 输入框，输入认证器中的 6 位验证码即可登录。

---

## 五、跟随上游更新（日常维护）

当 TriliumNext/Trilium 上游发布新版本时，按以下步骤同步：

### 1. 同步上游代码

```bash
# 首次：添加上游仓库
git remote add upstream https://github.com/TriliumNext/Trilium.git

# 每次更新：拉取并合并
git fetch upstream
git merge upstream/main

# 解决冲突（如有）
```

### 2. 补齐新增的中文翻译

合并后检查是否有新增的翻译 key：

```bash
# 在项目根目录运行
python3 -c "
import json
with open('apps/client/src/translations/en/translation.json') as f:
    en = json.load(f)
with open('apps/client/src/translations/cn/translation.json') as f:
    cn = json.load(f)
missing = [k for k in en if k not in cn]
print(f'缺失的翻译 key: {len(missing)}')
for k in sorted(missing):
    print(f'  {k}')
"
```

如有缺失，编辑 `apps/client/src/translations/cn/translation.json` 补齐中文翻译。

服务端同理，检查 `apps/server/src/assets/translations/cn/server.json`。

### 3. 更新版本号

- 更新 `1panel/apps/trilium/` 下的版本目录（如新增 `0.91.0/`）
- 更新 `docker-compose.yml` 中的镜像 tag

### 4. 构建新镜像

```bash
git tag v0.91.0
git push origin v0.91.0
```

CI 工作流会自动构建新版本镜像并推送到 `docker.1ms.run`。

### 5. 更新 1panel 中的容器

在 1panel 应用详情页 → 参数设置中更新镜像版本，重建容器。

---

## 六、环境变量说明

以下环境变量可在 1panel 安装时或 `docker-compose.yml` 中配置：

### 基础配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_DATA_DIR` | 数据存储路径 | `/home/node/trilium-data` |
| `TRILIUM_NETWORK_PORT` | 服务端口 | `8080` |
| `TRILIUM_SESSION_COOKIEMAXAGE` | 会话有效期（秒） | `1814400`（21天） |

### 安全配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_GENERAL_NOAUTHENTICATION` | 禁用认证（不推荐生产使用） | `false` |
| `TRILIUM_NETWORK_TRUSTEDREVERSEPROXY` | 信任反向代理头 | `false` |
| `TRILIUM_NETWORK_HTTPS` | 启用 HTTPS | `false` |

### CORS 配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_NETWORK_CORSALLOWORIGIN` | 允许的跨域来源 | 空 |
| `TRILIUM_NETWORK_CORSALLOWMETHODS` | 允许的跨域方法 | 空 |
| `TRILIUM_NETWORK_CORSALLOWHEADERS` | 允许的跨域头 | 空 |

### OAuth/SSO 配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_OAUTH_BASE_URL` | OAuth 回调地址 | 空 |
| `TRILIUM_OAUTH_CLIENT_ID` | OAuth 客户端 ID | 空 |
| `TRILIUM_OAUTH_CLIENT_SECRET` | OAuth 客户端密钥 | 空 |
| `TRILIUM_OAUTH_ISSUER_BASE_URL` | OAuth 发行方地址 | `https://accounts.google.com` |
| `TRILIUM_OAUTH_ISSUER_NAME` | OAuth 发行方名称 | `Google` |

---

## 七、反向代理配置

如使用 Nginx 反向代理访问 Trilium，需进行以下配置：

### Nginx 配置示例

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

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 信任反向代理

在 `docker-compose.yml` 的 `environment` 中添加：

```yaml
environment:
  - TRILIUM_NETWORK_TRUSTEDREVERSEPROXY=true
```

如果只信任特定 IP，可设置为具体地址：

```yaml
environment:
  - TRILIUM_NETWORK_TRUSTEDREVERSEPROXY=172.16.0.0/12
```

---

## 八、数据备份

Trilium 的所有数据存储在 `./data/trilium-data/` 目录中，包括：

- SQLite 数据库（`document.db`）
- 配置文件（`config.ini`）
- 日志文件
- 备份文件

建议定期备份该目录。1panel 支持定时备份功能，可在应用详情页配置。

---

## 九、常见问题

### Q: 忘记密码怎么办？

如果开启了 2FA，可使用恢复码登录。如果忘记密码且没有恢复码，只能重置密码（受保护笔记将丢失）。

### Q: 如何更新版本？

1. 同步上游代码（见第五节）
2. 补齐新增中文翻译
3. 推送新 tag 触发构建
4. 在 1panel 中重建容器

### Q: 镜像拉取失败？

检查 Registry 地址是否正确，确认网络可以访问 `docker.1ms.run`。如果使用国内服务器，可能需要配置 Docker 镜像加速器。

### Q: 如何在 1panel 中修改环境变量？

在 1panel 应用详情页 → 参数设置中修改，修改后需要重建容器生效。

### Q: 上游更新后中文翻译有缺失怎么办？

合并上游代码后运行第五节的检查脚本，将缺失的 key 翻译后填入 `cn/translation.json` 和 `cn/server.json` 即可。
