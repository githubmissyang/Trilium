# Trilium Notes

Trilium Notes 是一个层次化的笔记应用，专注于建立个人知识库。

## 主要特性

- 层次化的笔记结构，支持无限嵌套
- 富文本编辑（基于 CKEditor 5）
- 代码笔记，支持语法高亮
- 关系和属性系统，支持笔记间关联
- 笔记版本历史
- 强大的全文搜索
- 内置书签管理
- 脚本引擎，支持自动化
- 同步功能，多设备间数据同步
- 端到端加密的受保护笔记
- 内置 TOTP 双因子认证，保障访问安全
- 支持 OAuth/SSO 单点登录

## 使用说明

### 首次访问

安装后访问 `http://<服务器IP>:<端口>`，首次使用需设置登录密码。

### 开启双因子认证（2FA）

1. 登录后进入 **设置 → 多因子认证**
2. 开启 MFA 开关
3. 选择 **TOTP** 方式
4. 点击 **生成密钥**，使用 Google Authenticator / Microsoft Authenticator 等应用扫码
5. 点击 **生成恢复码**，妥善保存恢复码（用于丢失认证器时恢复访问）

### 反向代理配置

如使用 Nginx 等反向代理，需在环境变量中设置信任代理头：

```yaml
environment:
  - TRILIUM_NETWORK_TRUSTEDREVERSEPROXY=true
```

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TRILIUM_DATA_DIR` | 数据存储路径 | `/home/node/trilium-data` |
| `TRILIUM_NETWORK_PORT` | 服务端口 | `8080` |
| `TRILIUM_NETWORK_TRUSTEDREVERSEPROXY` | 信任反向代理头 | `false` |
| `TRILIUM_GENERAL_NOAUTHENTICATION` | 禁用认证（不推荐） | `false` |
| `TRILIUM_SESSION_COOKIEMAXAGE` | 会话有效期（秒） | `1814400`（21天） |
