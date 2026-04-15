# Custom Build 使用说明

[返回 README](../../README.md) | 中文 | [English](../en/custom-build.md)

如果你只是想 **快速做出一份自己的镜像**，最推荐的方式就是用 GitHub Actions 里的 **Custom Build**。

现在它采用的是显式组合模型：

- `base_system`
- `include_docker`
- `output_formats`

也就是说，你不是在选旧的 `variant` 名字，而是在明确地描述你要什么镜像。

---

## 3 分钟快速上手

### 第一步：打开 Actions

进入你自己的 fork 仓库后：

- 点击顶部 **Actions**
- 在左侧找到 **Custom Build**
- 点击 **Run workflow**

---

### 第二步：选基础组合

第一次使用，推荐直接选：

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

常见选择可以这样理解：

- `debian + false`：最通用，推荐第一次先用它
- `debian + true`：带 Docker 的 Debian 镜像
- `alpine + false`：更轻量
- `alpine + true`：更轻量，同时带 Docker

输出格式建议：

- `img`：最通用，测试和 raw 导盘都靠它
- `vmdk`：适合需要 VMDK 的场景
- `pve-ova`：适合 PVE 导入

如果你只是想先成功构建一版，**直接用 `debian + false + img`**。

---

### 第三步：按需填写参数

#### 情况 A：只改网络

如果你只是想改 LAN / DHCP 参数，可以填：

- `lan_server_ip=192.168.50.1`
- `lan_range_start=192.168.50.100`
- `lan_range_end=192.168.50.200`
- `lan_netmask=24`

#### 情况 B：同时改密码

如果你还想顺便改登录密码和 Web 管理账号，可以再填：

- `root_password=Passw0rd!234`
- `api_username=admin`
- `api_password=Adm1n!234`

#### 其他常见输入

- `landscape_version`
  - 指定要使用的 Landscape 版本
  - 留空时使用仓库默认值

当前优先级是：

**direct inputs > secrets > defaults**

---

### 第四步：点击运行

填完后点击：

- **Run workflow**

---

### 第五步：下载构建结果

等 workflow 跑完后：

- 打开这次运行记录
- 在页面下方找到 **Artifacts**
- 下载对应的构建产物

你拿到的通常会包含：

- raw 镜像 `.img`
- 构建元信息 `build-metadata.txt`
- 生效配置 `effective-landscape_init.toml`
- 如果你请求了额外格式，还会包含 `.vmdk` / `.ova`

---

## 如何选组合

### 我第一次应该怎么选？

建议：

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

### 我想要 Docker

把：

- `include_docker=true`

### 我想要更轻量

把：

- `base_system=alpine`

### 我想导入 PVE

把：

- `output_formats=img,pve-ova`

这样你同时拥有：

- 便于测试和兜底的 raw `.img`
- 直接导入用的 `.ova`

---

## 构建完成后还能做什么

如果你已经成功跑完一次 Custom Build，后面还可以继续用：

- **Test Image**

它适合做这些事：

- 对已有 artifact 再跑一次复测
- 补做 readiness / dataplane 验证
- 重新传 SSH / API 凭据测试

现在复测入口使用的是：

- `run_id`
- `artifact_id`

而不是旧的 `artifact_suffix` 命名。

---

## 常见问题

### 我第一次到底选哪个？

直接选：

- `base_system=debian`
- `include_docker=false`
- `output_formats=img`

### `pve-ova` 是不是会替代 `.img`？

不会。

推荐仍然保留 `img`，然后按需再加 `pve-ova`。

### dataplane 为什么有时不会跑？

规则是：

- `include_docker=false` → 跑 dataplane
- `include_docker=true` → 只跑 readiness

这不是按旧 variant 名字判断，而是按能力规则判断。

---

## 一句话建议

如果你的目标是：

> **“我想尽快做出一份自己的镜像。”**

那就先用：

- `debian + no-docker + img`

先把第一份镜像做出来，再决定要不要加 Docker、换 Alpine 或增加 `vmdk` / `pve-ova`。
