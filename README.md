# cangjie

[![Build Multi-Arch Images](https://github.com/ertu426/cangjie-docker/actions/workflows/build-images.yml/badge.svg)](https://github.com/ertu426/cangjie-docker/actions/workflows/build-images.yml)

多架构（`linux/amd64` + `linux/arm64`）容器镜像，基于 Debian **trixie**，预装仓颉（Cangjie）SDK，提供 `default` 与 `vite` 两种变体。

当前版本：**1.2.1**（以 `.github/workflows/build-images.yml` 中 `env.CANGJIE_VERSION` 为准）

---

## 镜像变体

| 变体 | 说明 | 基础镜像 |
|---|---|---|
| `default` | 标准版：预装 Cangjie SDK（`cjnative` 后端）+ 通用构建工具链 | `buildpack-deps:trixie` |
| `vite` | 在标准版基础上额外预装 [Node.js / Vite / 前端构建工具链] | `buildpack-deps:trixie` |

两个变体均同时提供 `linux/amd64` 与 `linux/arm64`，同一标签即多架构 manifest，Docker 拉取时自动匹配宿主架构。

---

## 支持的架构

| 架构 | Platform |
|---|---|
| x86_64 | `linux/amd64` |
| ARM64 | `linux/arm64` |

---

## 标签（Tags）

镜像发布在 GitHub Container Registry：

```
ghcr.io/ertu426/cangjie
```

> 下面示例中的 `1.2.1` 即当前 `env.CANGJIE_VERSION` 的值；发布新版本后所有版本标签自动跟随变化。

### `default` 变体（正式发布）

| 标签 | 含义 |
|---|---|
| `latest` | 最新正式发布版本 |
| `1.2.1` | 当前版本 |
| `v1.2.1` | 当前版本（带 `v` 前缀，与 git tag 风格一致） |
| `trixie` | 基于 trixie 的最新正式版本 |
| `trixie-1.2.1` | 基于 trixie 的当前版本 |
| `main` / `main-trixie` | `main` 分支最新构建（开发用，非稳定） |

### `vite` 变体（正式发布）

| 标签 | 含义 |
|---|---|
| `vite` | 最新正式发布版本 |
| `1.2.1-vite` | 当前版本 |
| `v1.2.1-vite` | 当前版本（带 `v` 前缀） |
| `trixie-vite` | 基于 trixie 的最新正式版本 |
| `trixie-1.2.1-vite` | 基于 trixie 的当前版本 |
| `main-vite` | `main` 分支最新构建（开发用，非稳定） |

> 每个标签都指向包含 `linux/amd64` 与 `linux/arm64` 的多架构 manifest，无需手动指定架构。

---

## 快速开始

### 拉取

```bash
# 标准版
docker pull ghcr.io/ertu426/cangjie:latest

# vite 变体
docker pull ghcr.io/ertu426/cangjie:vite

# 精确版本
docker pull ghcr.io/ertu426/cangjie:trixie-1.2.1
```

### 运行

```bash
docker run -it --rm ghcr.io/ertu426/cangjie:latest bash
docker run -it --rm ghcr.io/ertu426/cangjie:vite bash
```

### 在 docker-compose 中使用

```yaml
services:
  app:
    image: ghcr.io/ertu426/cangjie:latest
    volumes:
      - ./src:/workspace
    working_dir: /workspace
    command: bash
```

### 在 GitHub Actions 中使用

```yaml
- name: Log in to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Run
  run: docker run --rm ghcr.io/ertu426/cangjie:latest [your-command]
```

---

## 镜像内容

### 环境变量

| 变量 | 值 | 说明 |
|---|---|---|
| `CANGJIE_HOME` | `/opt/cangjie` | Cangjie SDK 根目录 |
| `LD_LIBRARY_PATH` | `/opt/cangjie/runtime/lib/linux_<arch>_cjnative:/opt/cangjie/tools/lib` | 运行时库搜索路径 |
| `PATH` | `/opt/cangjie/bin:/opt/cangjie/tools/bin:$PATH:/root/.cjpm/bin` | 包含 `cjc`、`cjpm` 等工具 |
| `WORKDIR` | `/workspace` | 默认工作目录 |

### 预装软件

- Cangjie SDK（`cjnative` 后端），版本见 `env.CANGJIE_VERSION`
- Python 3.11 运行时（由 SDK 工具链依赖）
- [默认额外软件，如 curl / git / build-essential 等]

---

## 从源码构建

本仓库的 Dockerfile 采用 `ARG` 参数 + `RUN curl + sha256sum` 方式按架构选择 SDK。本地构建时需显式传入参数：

```bash
# amd64
docker build \
  --build-arg ARCH=amd64 \
  --build-arg SDK_ARCH=x86_64 \
  --build-arg SDK_URL="https://cangjie-lang.cn/v1/files/auth/downLoad?nsId=142267&fileName=cangjie-sdk-linux-x64-1.2.1.tar.gz&objectKey=6af2edbae4bd101f57762f7c" \
  --build-arg SDK_SHA256="5616624fcb48db150df2accde6669a2c8cc5e851d79e623a5469c4e4cd7a491f" \
  --build-arg CANGJIE_VERSION=1.2.1 \
  -f default/Dockerfile -t cangjie:local-amd64 .

# aarch64（需 buildx + QEMU）
docker buildx build --platform linux/arm64 \
  --build-arg ARCH=aarch64 \
  --build-arg SDK_ARCH=aarch64 \
  --build-arg SDK_URL="https://cangjie-lang.cn/v1/files/auth/downLoad?nsId=142267&fileName=cangjie-sdk-linux-aarch64-1.2.1.tar.gz&objectKey=6af2edbbe4bd101f57762f7d" \
  --build-arg SDK_SHA256="5c0f58166f287d7d1317eca416446bcdfd22d0fe8bfc62c38de3e4bfda8b791d" \
  --build-arg CANGJIE_VERSION=1.2.1 \
  -f default/Dockerfile -t cangjie:local-arm64 .
```

`vite/Dockerfile` 同理，把 `-f default/Dockerfile` 改成 `-f vite/Dockerfile`。

---

## 项目结构

```
.
├── .github/
│   └── workflows/
│       └── build-images.yml    # 多架构构建 + GHCR 推送
├── default/
│   └── Dockerfile              # 标准版，单文件支持 amd64/aarch64
├── vite/
│   └── Dockerfile              # vite 变体，单文件支持 amd64/aarch64
└── README.md
```

Dockerfile 通过 `ARG` 参数按架构选择 SDK：

| 参数 | `amd64` | `aarch64` |
|---|---|---|
| `ARCH` | `amd64` | `aarch64` |
| `SDK_ARCH` | `x86_64` | `aarch64` |
| `SDK_URL` | `...cangjie-sdk-linux-x64-${CANGJIE_VERSION}.tar.gz...` | `...cangjie-sdk-linux-aarch64-${CANGJIE_VERSION}.tar.gz...` |
| `SDK_SHA256` | `5616...` | `5c0f...` |
| `CANGJIE_VERSION` | `1.2.1` | `1.2.1` |

---

## CI / CD

`.github/workflows/build-images.yml` 的行为：

| 触发 | 结果 |
|---|---|
| **Pull Request** | 只构建验证（`push: false`），不产生镜像 |
| **push 到 `main`** | 构建并推送开发标签：`main` / `main-trixie` / `main-vite` |
| **push tag `v*`** | 构建并推送全套正式标签；**要求 tag 名 = `v${env.CANGJIE_VERSION}`**，否则构建直接失败 |
| **workflow_dispatch** | 构建并推送全套正式标签（等同发布） |

### 前置条件

1. 仓库 **Settings → Actions → General → Workflow permissions** 选择 **Read and write**（允许 workflow 推 GHCR）；
2. 首次推送后，到 **Packages → cangjie → Package settings** 按需调整包可见性（默认 private）。

### 发布新版本

1. 修改 `.github/workflows/build-images.yml` 中的 `env.CANGJIE_VERSION`（如有必要同步 `SDK_*_URL` / `SDK_*_SHA256`）；
2. 提交并推送：

   ```bash
   git add .github/workflows/build-images.yml
   git commit -m "release v1.3.0"
   git push origin main

   git tag v1.3.0
   git push origin v1.3.0
   ```

3. 若 tag 名与 `env.CANGJIE_VERSION` 不一致（例如 tag 是 `v1.3.0` 但 env 还是 `1.2.1`），build job 会在 **Verify tag matches CANGJIE_VERSION** 步骤直接失败并提示修正。

发布 `v1.2.1` 后产生的镜像标签：

```
ghcr.io/ertu426/cangjie:latest
ghcr.io/ertu426/cangjie:1.2.1
ghcr.io/ertu426/cangjie:v1.2.1
ghcr.io/ertu426/cangjie:trixie
ghcr.io/ertu426/cangjie:trixie-1.2.1
ghcr.io/ertu426/cangjie:vite
ghcr.io/ertu426/cangjie:1.2.1-vite
ghcr.io/ertu426/cangjie:v1.2.1-vite
ghcr.io/ertu426/cangjie:trixie-1.2.1-vite
```

> ⚠️ 每次发布都会把 `latest` / `trixie` / `vite` 刷新为当前 `env.CANGJIE_VERSION`。若需要引用历史版本，请使用精确版本号标签（如 `v1.0.0`、`trixie-1.0.0`、`v1.0.0-vite`）。

---

## 许可证

MIT