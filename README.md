# Landscape Mini

[![Latest Release](https://img.shields.io/github/v/release/Cloud370/landscape-mini)](https://github.com/Cloud370/landscape-mini/releases/latest)

[English](docs/en/README.md) | 中文 | [贡献流程](CONTRIBUTING.md) | [**下载最新镜像**](https://github.com/Cloud370/landscape-mini/releases/latest)

Landscape Router 的最小化 x86 镜像构建器。支持 **Debian Trixie** 和 **Alpine Linux** 两种基础系统，可生成精简磁盘镜像，并支持 BIOS + UEFI 双启动。

上游项目：[Landscape Router](https://github.com/ThisSeanZhang/landscape)

## 先看这里

如果你只是想**快速体验 Landscape**：

- 直接去 [Release 页面下载预编译镜像](https://github.com/Cloud370/landscape-mini/releases/latest)

如果你想**做一份自己的镜像**，例如修改：

- LAN / DHCP 网段
- Linux 登录密码
- Web 管理用户名和密码

推荐优先使用：

- [Custom Build 使用说明](docs/zh/custom-build.md)

如果你要开发或调试构建系统本身，再看下面的本地构建说明。

## 特性

- 同时支持 Debian 和 Alpine 两种基础系统
- 镜像身份由显式组合定义：`base_system + include_docker + output_formats`
- 输出格式支持 `img`、`vmdk`、`pve-ova`
- 支持 BIOS + UEFI，常见虚拟化环境都比较友好
- fork 用户也可以直接在 GitHub 上跑自定义构建
- GitHub Actions 已经接好，支持自动构建、测试和发布

## 本地开发

如果你已经确定要在本地构建、调试或验证未推送改动，再看这里。

### 本地构建

```bash
# 安装构建依赖（首次）
make deps

# 默认组合：debian + no-docker + img
make build

# Alpine raw image
make build BASE_SYSTEM=alpine

# Debian + Docker + img,pve-ova
make build INCLUDE_DOCKER=true OUTPUT_FORMATS=img,pve-ova

# Alpine + Docker + img,vmdk,pve-ova
make build BASE_SYSTEM=alpine INCLUDE_DOCKER=true OUTPUT_FORMATS=img,vmdk,pve-ova
```

### 本地测试

```bash
# 自动化 readiness 检查（无需交互）
make deps-test
make test

# 数据面测试仅适用于 include_docker=false 的 raw image
make test-dataplane

# 也可以直接指定任意 raw image
./tests/test-readiness.sh output/landscape-mini-x86-alpine.img
./tests/test-dataplane.sh output/landscape-mini-x86-debian.img

# 交互式启动（串口控制台）
make test-serial
```

## 部署

### 物理机 / U 盘

```bash
dd if=output/landscape-mini-x86-debian.img of=/dev/sdX bs=4M status=progress
```

### Proxmox VE (PVE)

推荐两种方式：

#### 方式 1：直接使用 `pve-ova`

- 构建时包含 `pve-ova` 输出格式
- 上传生成的 `.ova`
- 在 PVE 中导入后检查启动模式、磁盘控制器和 bridge 绑定

#### 方式 2：使用 raw `.img`

1. 上传镜像到 PVE 服务器
2. 创建虚拟机（不添加磁盘）
3. 导入磁盘：`qm importdisk <vmid> landscape-mini-x86-debian.img local-lvm`
4. 在 VM 硬件设置中挂载导入的磁盘
5. 设置启动顺序，启动虚拟机

### 云服务器（dd 脚本）

使用 [reinstall](https://github.com/bin456789/reinstall) 脚本将自定义镜像写入云服务器：

```bash
bash <(curl -sL https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh) \
    dd --img='https://github.com/Cloud370/landscape-mini/releases/latest/download/landscape-mini-x86-debian.img.gz'
```

> 根分区会在首次启动时自动扩展以填满整个磁盘，无需手动操作。

## 默认凭据

| 场景 | 用户 | 密码 |
|------|------|------|
| SSH / 系统登录 | `root` | `landscape` |
| SSH / 系统登录 | `ld` | `landscape` |
| 管理网页 | `root` | `root` |

> `custom-build.yml` 支持通过 workflow inputs 或 GitHub Secrets 覆盖 Linux / Web 管理凭据。明文 inputs 适合个人临时使用；如果你更在意安全，优先使用 `CUSTOM_ROOT_PASSWORD`、`CUSTOM_API_USERNAME`、`CUSTOM_API_PASSWORD`。

## 构建配置

编辑 `build.env` 或通过环境变量覆盖：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `BASE_SYSTEM` | `debian` | 基础系统：`debian` / `alpine` |
| `INCLUDE_DOCKER` | `false` | 是否包含 Docker：`true` / `false` |
| `OUTPUT_FORMATS` | `img` | 输出格式列表：`img`、`vmdk`、`pve-ova`，逗号分隔 |
| `APT_MIRROR` | _(auto probe)_ | Debian 软件源显式覆盖；如果为空则从候选列表自动探测 |
| `ALPINE_MIRROR` | _(auto probe)_ | Alpine 软件源显式覆盖；如果为空则从候选列表自动探测 |
| `DOCKER_APT_MIRROR` | _(auto probe)_ | Debian Docker APT 仓库显式覆盖；如果为空则从候选列表自动探测 |
| `DOCKER_APT_GPG_URL` | _(auto probe)_ | Debian Docker APT GPG key 显式覆盖；如果为空则从候选列表自动探测 |
| `LANDSCAPE_VERSION` | `v0.18.2` | 上游 Landscape 版本号 |
| `LANDSCAPE_REPO` | `https://github.com/ThisSeanZhang/landscape` | 上游 Landscape 发布仓库 |
| `IMAGE_SIZE_MB` | `2048` | 初始镜像大小（最终会自动缩小） |
| `ROOT_PASSWORD` | `landscape` | root / ld 登录密码 |
| `TIMEZONE` | `Asia/Shanghai` | 时区 |
| `LOCALE` | `C.UTF-8` | 系统 locale |

### 自定义构建（GitHub Actions）

仓库提供 `custom-build.yml`，面向 fork 用户提供组合式构建入口，支持：

- `base_system`：`debian` / `alpine`
- `include_docker`：`true` / `false`
- `output_formats`
- `landscape_version`
- `lan_server_ip` / `lan_range_start` / `lan_range_end` / `lan_netmask`
- `root_password`
- `api_username` / `api_password`

当前优先级：**direct inputs > secrets > defaults**。

workflow 会把以下身份信息写入 `build-metadata.txt`：

- `base_system`
- `include_docker`
- `output_formats`
- `produced_files`
- `artifact_id`

网络拓扑的有效配置会随 artifact 一起携带为 `effective-landscape_init.toml`，供 `test.yml` 和 release promotion 校验使用。

## 自动化测试

### Readiness 检查

`make test` 或 `./tests/test-readiness.sh <image.img>` 执行统一的 router readiness 契约验证：

1. 复制镜像到临时文件（保护构建产物）
2. 后台启动 QEMU（自动检测 KVM）
3. 等待 SSH、API listener、API login、layout 检测完成
4. 验证 `eth0` / `eth1` 以及核心服务进入 running
5. 当 `include_docker=true` 时额外验证 Docker 可用
6. 输出 readiness / service / diagnostics 快照并清理 QEMU

### Dataplane 测试

`make test-dataplane` 或 `./tests/test-dataplane.sh <image.img>` 使用双 VM 拓扑验证真实 client-visible dataplane：

```text
Router VM (eth0=WAN/SLIRP, eth1=LAN/mcast) ←→ Client VM (CirrOS, eth0=mcast)
```

测试项：DHCP 分配、Router API 中 lease 可见、Router ↔ Client LAN 连通。

> dataplane 调度规则基于 `include_docker=false`，而不是旧的 variant 名称。

## CI/CD

- **CI**：手动触发始终可用；对 push 到 `main` / PR，仅在构建相关文件、`.github/` 下相关文件或 `CHANGELOG.md` 变更时自动运行
- **构建矩阵**：`debian/alpine × include_docker=true/false`
- **Readiness / E2E 覆盖**：四个组合都跑 readiness；只有 `include_docker=false` 的组合跑 dataplane
- **Artifact contract**：每个 image artifact 都包含 raw `.img`、`build-metadata.txt`、`effective-landscape_init.toml`，并可按请求附带 `.vmdk` / `.ova`
- **Custom Build**：`custom-build.yml` 支持 fork 用户按显式组合构建，并支持 LAN/DHCP、Linux 密码、Web 管理账号密码输入
- **Manual Retest**：`test.yml` 支持按 `run_id` 或 `artifact_id` 复测一整组 CI artifacts，并重新传入 SSH / API 凭据
- **Release**：推送 `v*` tag 时，`release.yml` 仅 promote 同一 commit 上已通过 CI 的 artifact，校验 tuple metadata 后创建 GitHub Release

## 许可证

本项目采用 **GPL-3.0** 协议发布，并与上游 [Landscape Router](https://github.com/ThisSeanZhang/landscape) 保持一致。
