# Docker 镜像同步仓库

利用 GitHub Actions 将海外 Docker 镜像（Docker Hub 等）同步到 **GHCR (GitHub Container Registry)**，方便在国内网络环境下拉取。

## 原理

```
Docker Hub (海外)  ──GitHub Actions──▶  ghcr.io/<你的用户名>/<镜像名>
        │                                        │
   国内访问困难                            国内可正常拉取 ✅
```

GitHub Actions 运行在海外服务器上，可以正常访问 Docker Hub。同步后的镜像托管在 `ghcr.io`，国内可通过代理或直连拉取。

## 使用方法

### 1. 修改镜像列表

编辑仓库根目录的 `images.txt`，每行一个镜像，格式与 `docker pull` 一致：

```
nginx:1.25.3
bitnami/redis:7.2
library/alpine:3.19
```

> 官方镜像（如 `nginx`、`alpine`）可省略 `library/` 前缀，脚本会自动补全。

### 2. 手动触发同步

进入仓库 **Actions** 页面 → 选择 **Sync Docker Images** 工作流 → 点击 **Run workflow**。

可以在输入框中临时填写要同步的镜像（每行一个），留空则使用 `images.txt`。

### 3. 定时同步（可选）

工作流默认每天 UTC 00:00（北京时间 08:00）自动运行，保持镜像为最新版本。可在 `sync-images.yml` 中修改 `schedule.cron`。

### 4. 拉取镜像

同步完成后，在国内拉取：

```bash
docker pull ghcr.io/<你的用户名>/nginx:1.25.3
```

例如本仓库所有者：

```bash
docker pull ghcr.io/lin285170/nginx:1.25.3
docker pull ghcr.io/lin285170/bitnami-redis:7.2
```

## 镜像命名规则

| 源镜像 | 同步到 GHCR |
|--------|------------|
| `nginx:1.25` | `ghcr.io/<owner>/nginx:1.25` |
| `library/nginx:1.25` | `ghcr.io/<owner>/nginx:1.25` |
| `bitnami/redis:7.2` | `ghcr.io/<owner>/bitnami-redis:7.2` |
| `grafana/grafana:10.2` | `ghcr.io/<owner>/grafana-grafana:10.2` |

多层级路径用 `-` 连接，避免与 GHCR 单层命名空间冲突。

## 支持多架构

使用 `docker buildx imagetools` 同步完整的多架构清单（manifest list），拉取时会自动匹配当前主机架构（amd64 / arm64）。

## 权限说明

工作流使用内置的 `GITHUB_TOKEN` 登录 GHCR，无需额外配置 Secrets。首次同步后，GHCR 镜像包默认为 **private**，如需公开拉取，到 GitHub → Packages → 对应包 → Settings → Change visibility 设为 Public。

## 目录结构

```
.
├── .github/workflows/
│   └── sync-images.yml    # GitHub Actions 同步工作流
├── images.txt             # 待同步镜像列表
└── README.md
```
