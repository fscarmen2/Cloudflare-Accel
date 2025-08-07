# Cloudflare-Accel

是一个基于 Cloudflare Workers 的代理服务，旨在加速 GitHub 文件下载和 Docker 镜像拉取。通过 Cloudflare 的全球边缘网络，提供更快、更稳定的下载体验。项目提供直观的网页界面，支持将 GitHub 文件链接和 Docker 镜像地址转换为加速链接或命令，并自动复制到剪贴板。

## 目录

- [特点](#特点)
- [部署方法](#部署方法)
- [参数说明](#参数说明)
- [使用示例](#使用示例)
- [许可证](#许可证)

## 特点

- ⚡ GitHub 文件加速
- 🐳 Docker 镜像加速
- 🎨 现代化 UI
- 🔒 白名单控制

## 部署方法

1. **创建 Cloudflare Worker**：
   - 登录 [Cloudflare 仪表板](https://dash.cloudflare.com/)。
   - 转到 Workers 部分，点击“创建 Worker”。
   - 将 `worker.js` 代码（见项目仓库）粘贴到 Worker 编辑器。
   - 点击“部署”按钮，Worker 将上线。

2. **配置白名单（可选）**：
   - 修改 `ALLOWED_HOSTS` 和 `ALLOWED_PATHS` 数组，添加允许的域名和路径。
   - 设置 `RESTRICT_PATHS = true` 启用路径限制，仅允许 `ALLOWED_PATHS` 中的路径。

3. **添加 Worker 路由，自定义域名**：
   - Settings → Domains & Routes → Add → Custom domain
   - 输入自定义域名，保存。

## 参数说明

| 参数名            | 说明                                                                 | 默认值                                                                 |
|-------------------|----------------------------------------------------------------------|----------------------------------------------------------------------|
| `ALLOWED_HOSTS`   | 允许代理的域名列表（默认白名单），未列出的域名将返回 400 错误       | `['quay.io', 'gcr.io', 'k8s.gcr.io', 'registry.k8s.io', 'ghcr.io', 'docker.cloudsmith.io', 'registry-1.docker.io', 'github.com', 'api.github.com', 'raw.githubusercontent.com', 'gist.githubusercontent.com']` |
| `RESTRICT_PATHS`  | 是否限制 GitHub 和 Docker 请求的路径，`true` 要求路径匹配 `ALLOWED_PATHS`，`false` 允许所有路径 | `false`                                                              |
| `ALLOWED_PATHS`   | 允许的 GitHub 和 Docker 路径关键字，仅当 `RESTRICT_PATHS = true` 时生效 | `['library', 'user-id-1', 'user-id-2']`                             |

### 修改白名单
- **添加新域名**：编辑 `ALLOWED_HOSTS`，如添加 `docker.io`：
  ```javascript
  const ALLOWED_HOSTS = [...ALLOWED_HOSTS, 'docker.io'];
  ```
- **添加新路径**：编辑 `ALLOWED_PATHS`，如添加 `user-id-3`：
  ```javascript
  const ALLOWED_PATHS = [...ALLOWED_PATHS, 'user-id-3'];
  ```
- **启用路径限制**：设置 `RESTRICT_PATHS = true`，确保 `ALLOWED_PATHS` 包含所需路径。

## 使用示例

1. **访问首页**：
   ```bash
   curl https://your-domain/
   ```
   - 显示网页，包含 GitHub 和 Docker 输入框，右上角主题切换按钮，黄色闪电 favicon。

2. **GitHub 文件加速**：
   - 输入：`https://raw.githubusercontent.com/user-id-1/repo/file`
   - 输出：`https://your-domain/raw.githubusercontent.com/user-id-1/repo/file`
   - 自动复制，弹窗提示“已复制到剪贴板”，显示 📋 复制 和 🔗 打开 按钮。
   - 测试（`RESTRICT_PATHS = true`）：
     ```bash
     wget -O /dev/null https://your-domain/raw.githubusercontent.com/user-id-1/repo/file  # 成功
     wget -O /dev/null https://your-domain/raw.githubusercontent.com/other-user/repo/file  # 返回 403: Error: The path is not in the allowed paths.
     ```
   - 测试（`RESTRICT_PATHS = false`）：
     ```bash
     wget -O /dev/null https://your-domain/raw.githubusercontent.com/other-user/repo/file  # 成功
     ```

3. **Docker 镜像加速**：
   - 输入：`nginx` 或 `ghcr.io/user-id-1/hubproxy`
   - 输出：`docker pull your-domain/nginx`
   - 自动复制，弹窗提示“已复制到剪贴板”，显示 📋 复制 按钮。
   - 测试（`RESTRICT_PATHS = true`）：
     ```bash
     docker pull your-domain/nginx  # 成功（library）
     docker pull your-domain/ghcr.io/user-id-1/hubproxy  # 成功
     docker pull your-domain/ghcr.io/unknown/hubproxy  # 返回 403: Error: The path is not in the allowed paths.
     ```
   - 测试（`RESTRICT_PATHS = false`）：
     ```bash
     docker pull your-domain/ghcr.io/unknown/hubproxy  # 成功
     ```

4. **白名单外域名**：
   ```bash
   wget -O /dev/null https://your-domain/invalid.com/path
   ```
   - 返回：`Error: Invalid target domain.`

## 许可证

本项目基于 MIT 许可证。详情见 [LICENSE](LICENSE) 文件。