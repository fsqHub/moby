# Docker 构建过程的网络需求详解

## 简短回答

**是的，构建 Docker 二进制需要网络连接**，主要用于：

1. ✅ 拉取基础容器镜像
2. ✅ 下载 Go 源码和依赖包
3. ✅ 克隆依赖组件（runc、containerd、libnetwork）
4. ✅ 安装系统包（apt/yum）

---

## 详细网络需求分析

### 阶段 1：构建开发环境镜像（`make build`）

#### 1.1 拉取基础镜像

```bash
# Dockerfile.aarch64 的第一行
FROM aarch64/ubuntu:wily

# 需要从 Docker Hub 拉取
docker pull aarch64/ubuntu:wily
# 大小: 约 120-150 MB
```

**网络访问**：
- 目标：`registry-1.docker.io` (Docker Hub)
- 协议：HTTPS (443)
- 可替代：使用镜像加速器（阿里云、腾讯云等）

#### 1.2 安装系统包（apt-get）

```bash
# Dockerfile.aarch64 中
RUN apt-get update && apt-get install -y \
    apparmor \
    aufs-tools \
    build-essential \
    curl \
    git \
    ...
```

**网络访问**：
- 目标：`archive.ubuntu.com`、`ports.ubuntu.com`（ARM64）
- 协议：HTTP/HTTPS
- 流量：约 200-500 MB
- 可替代：使用本地/内网 apt 镜像

#### 1.3 下载 Go 源码

```bash
# Dockerfile.aarch64 中
RUN curl -fsSL https://golang.org/dl/go${GO_VERSION}.src.tar.gz | tar ...
```

**网络访问**：
- 目标：`golang.org` (Google)
- 文件：`go1.7.5.src.tar.gz`
- 大小：约 15 MB
- 可替代：使用国内镜像（如 `https://golang.google.cn/dl/`）

#### 1.4 克隆依赖组件

通过 `hack/dockerfile/install-binaries.sh` 脚本：

```bash
# runc
git clone https://github.com/docker/runc.git
# 大小: 约 3-5 MB

# containerd
git clone https://github.com/docker/containerd.git
# 大小: 约 5-10 MB

# libnetwork (docker-proxy)
git clone https://github.com/docker/libnetwork.git
# 大小: 约 2-3 MB
```

**网络访问**：
- 目标：`github.com`
- 协议：HTTPS (443) 或 Git (9418)
- 总流量：约 10-20 MB
- 可替代：使用 Gitee 镜像或本地 Git 缓存

#### 1.5 Go 依赖包（vendor）

v1.13.1 使用 vendor 机制，依赖包已内置在源码中：

```bash
ls vendor/
# github.com/
# golang.org/
# ...
```

**网络访问**：
- ✅ **不需要**（依赖已 vendor 化）
- ⚠️ 如果缺失，会从网络下载（`go get`）

---

### 阶段 2：编译二进制（`make binary`）

#### 2.1 编译主程序

```bash
# 在构建容器内执行
go build -o dockerd ./cmd/dockerd
go build -o docker ./cmd/docker
```

**网络访问**：
- ✅ **通常不需要**（vendor 已包含依赖）
- ⚠️ 除非 vendor 不完整

#### 2.2 编译依赖组件

```bash
# runc
cd $GOPATH/src/github.com/opencontainers/runc
make static

# containerd
cd $GOPATH/src/github.com/docker/containerd
make
```

**网络访问**：
- ✅ **不需要**（已克隆到本地）

---

### 阶段 3：构建 RPM 包（`make rpm`）

#### 3.1 拉取 RPM 构建镜像

```bash
# 根据 contrib/builder/rpm/aarch64/centos-7/Dockerfile
FROM arm64v8/centos:7

docker pull arm64v8/centos:7
# 大小: 约 200 MB
```

**网络访问**：
- 目标：Docker Hub
- 流量：200 MB

#### 3.2 安装 RPM 构建依赖

```bash
RUN yum install -y \
    rpm-build \
    systemd-devel \
    ...
```

**网络访问**：
- 目标：`mirror.centos.org`、`vault.centos.org`
- 流量：约 100-300 MB

---

## 总网络流量估算

### 首次完整构建（从零开始）

| 阶段 | 组件 | 大小 |
|------|------|------|
| 基础镜像 | aarch64/ubuntu:wily | 120 MB |
| 系统包 | apt-get install | 300 MB |
| Go 源码 | go1.7.5.src.tar.gz | 15 MB |
| Git 仓库 | runc + containerd + libnetwork | 20 MB |
| RPM 镜像 | arm64v8/centos:7 | 200 MB |
| RPM 依赖 | yum install | 150 MB |
| **总计** | | **约 800 MB** |

### 增量构建（已有镜像）

| 阶段 | 组件 | 大小 |
|------|------|------|
| Git 拉取 | 更新依赖仓库 | < 5 MB |
| 编译 | 本地操作 | 0 MB |
| **总计** | | **< 5 MB** |

---

## 离线构建方案

### 方案 1：提前下载镜像和依赖

#### 在有网环境准备

```bash
# 1. 拉取并保存镜像
docker pull aarch64/ubuntu:wily
docker pull arm64v8/centos:7
docker save -o docker-images-aarch64.tar aarch64/ubuntu:wily arm64v8/centos:7

# 2. 克隆 Git 仓库
mkdir -p ~/docker-deps
cd ~/docker-deps
git clone --bare https://github.com/docker/runc.git
git clone --bare https://github.com/docker/containerd.git
git clone --bare https://github.com/docker/libnetwork.git

# 3. 下载 Go 源码
wget https://golang.org/dl/go1.7.5.src.tar.gz

# 4. 打包传输
tar -czf docker-build-deps.tar.gz docker-images-aarch64.tar go1.7.5.src.tar.gz ~/docker-deps/
```

#### 在离线环境使用

```bash
# 1. 解压依赖包
tar -xzf docker-build-deps.tar.gz

# 2. 加载镜像
docker load -i docker-images-aarch64.tar

# 3. 配置本地 Git 仓库
export RUNC_REPO=/path/to/runc.git
export CONTAINERD_REPO=/path/to/containerd.git
export LIBNETWORK_REPO=/path/to/libnetwork.git

# 4. 修改 install-binaries.sh 使用本地仓库
vim hack/dockerfile/install-binaries.sh
# 将 git clone https://github.com/... 改为 git clone file:///path/to/...

# 5. 构建
make build
make binary
```

---

### 方案 2：使用内网镜像源

#### 配置 Docker 镜像加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
EOF

sudo systemctl restart docker
```

#### 配置 apt/yum 镜像源

```bash
# 对于 Ubuntu 基础镜像，修改 Dockerfile
RUN sed -i 's|archive.ubuntu.com|mirrors.ustc.edu.cn|g' /etc/apt/sources.list

# 对于 CentOS 基础镜像
RUN sed -i 's|mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-*.repo && \
    sed -i 's|#baseurl=http://mirror.centos.org|baseurl=https://mirrors.ustc.edu.cn|g' /etc/yum.repos.d/CentOS-*.repo
```

#### 配置 Go 代理

```bash
# 在 Dockerfile 中添加
ENV GOPROXY=https://goproxy.cn,direct
ENV GO111MODULE=off  # v1.13.1 使用 vendor，不需要 module
```

#### 配置 Git 使用镜像

```bash
# 将 GitHub 替换为 Gitee 镜像（如果有）
# 或设置 Git HTTP 代理
git config --global http.proxy http://proxy.example.com:8080
```

---

### 方案 3：完全离线构建容器

如果无法配置镜像源，可以制作离线构建容器：

```bash
# 在有网环境：

# 1. 构建完整开发环境
cd moby-1.13.1
make build

# 2. 保存构建镜像
docker save -o docker-dev-aarch64.tar docker:1.13.1-dev

# 3. 传输到离线环境
scp docker-dev-aarch64.tar offline-machine:/tmp/

# 在离线环境：

# 4. 加载镜像
docker load -i /tmp/docker-dev-aarch64.tar

# 5. 使用镜像编译（跳过 make build）
docker run -v $(pwd):/go/src/github.com/docker/docker \
    docker:1.13.1-dev \
    hack/make.sh binary

# 6. 查看产物
ls bundles/1.13.1/binary-daemon/
```

---

## 网络故障排查

### 问题 1：拉取镜像超时

```bash
# 症状
docker build -t docker -f Dockerfile.aarch64 .
# 输出: Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: TLS handshake timeout

# 解决
# 1. 使用镜像加速器（见上文）
# 2. 或手动拉取后重新 tag
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/ubuntu:wily
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/ubuntu:wily aarch64/ubuntu:wily
```

### 问题 2：apt-get update 失败

```bash
# 症状
Step 5/50 : RUN apt-get update
# 输出: Err http://archive.ubuntu.com/ubuntu wily Release.gpg
#       Could not resolve 'archive.ubuntu.com'

# 解决
# 在 Dockerfile 中添加（紧接 FROM 后）
RUN echo "nameserver 8.8.8.8" > /etc/resolv.conf && \
    sed -i 's|archive.ubuntu.com|mirrors.ustc.edu.cn|g' /etc/apt/sources.list
```

### 问题 3：git clone 失败

```bash
# 症状
fatal: unable to access 'https://github.com/docker/runc.git/': 
Failed to connect to github.com port 443: Connection timed out

# 解决方案 1: 使用代理
export http_proxy=http://proxy.example.com:8080
export https_proxy=http://proxy.example.com:8080

# 解决方案 2: 修改为 git:// 协议（如果防火墙允许）
# 编辑 hack/dockerfile/install-binaries.sh
git clone git://github.com/docker/runc.git ...

# 解决方案 3: 使用 Gitee 镜像（如果有）
git clone https://gitee.com/mirrors/runc.git ...
```

### 问题 4：Go 下载失败

```bash
# 症状
curl: (7) Failed to connect to golang.org port 443: Connection timed out

# 解决
# 修改 Dockerfile.aarch64 中的 Go 下载 URL
RUN mkdir /usr/src/go && \
    curl -fsSL https://golang.google.cn/dl/go${GO_VERSION}.src.tar.gz | tar ...
# 或使用国内镜像
RUN mkdir /usr/src/go && \
    curl -fsSL https://mirrors.ustc.edu.cn/golang/go${GO_VERSION}.src.tar.gz | tar ...
```

---

## 最小网络需求场景

### 场景：已有所有镜像和依赖

如果满足以下条件，可以**几乎不需要网络**：

```bash
# 条件检查清单
1. ✅ docker images | grep "aarch64/ubuntu.*wily"
2. ✅ docker images | grep "arm64v8/centos.*7"
3. ✅ ls vendor/ | wc -l  # vendor 目录完整（> 100 个包）
4. ✅ 修改了 install-binaries.sh 使用本地 Git 仓库

# 此时执行构建
make build   # 使用本地镜像，不拉取
make binary  # 使用 vendor 依赖，不下载
```

**网络访问**：
- ✅ 仅用于 apt/yum 更新包列表（可禁用）
- 流量：< 1 MB

---

## 推荐构建策略

### 策略 1：云服务器构建（有网环境）

**优点**：
- ✅ 网络通畅，无需配置
- ✅ 构建速度快
- ✅ 适合首次构建

**成本**：
- 流量费用：约 1 GB = 0.8-1.2 元（按量计费）

---

### 策略 2：离线环境 + 预准备依赖

**优点**：
- ✅ 适合生产环境（网络受限）
- ✅ 可重复构建
- ✅ 无外网依赖风险

**准备工作**：
- 在有网环境下载所有依赖（约 1 小时）
- 打包传输到离线环境

---

### 策略 3：内网镜像源

**优点**：
- ✅ 平衡网络需求和便利性
- ✅ 适合团队协作
- ✅ 加速重复构建

**配置要求**：
- Docker 镜像加速器
- apt/yum 内网源
- Git 镜像或代理

---

## 快速检查网络连通性

```bash
# 检查 Docker Hub
curl -I https://registry-1.docker.io/v2/
# 预期: HTTP/2 200

# 检查 GitHub
curl -I https://github.com
# 预期: HTTP/2 200

# 检查 golang.org
curl -I https://golang.org/dl/
# 预期: HTTP/2 200

# 检查 Ubuntu 软件源
curl -I http://ports.ubuntu.com/
# 预期: HTTP/1.1 200 OK

# 检查 CentOS 软件源
curl -I http://mirror.centos.org/
# 预期: HTTP/1.1 200 OK
```

---

## 总结

### 网络需求

| 阶段 | 是否需要网络 | 流量 | 可离线化 |
|------|-------------|------|----------|
| **make build** | ✅ 必须 | 500-700 MB | ⚠️ 需预准备 |
| **make binary** | ❌ 通常不需要 | < 5 MB | ✅ 易离线 |
| **make rpm** | ✅ 必须 | 200-400 MB | ⚠️ 需预准备 |

### 推荐方案

```
有网环境（云服务器）
    → 直接构建，无需配置
    → 总流量约 800 MB

受限网络（企业内网）
    → 配置内网镜像源
    → 减少 80% 流量

完全离线（高安全环境）
    → 预准备所有依赖
    → 制作离线构建容器
```

### 关键提示

⚠️ **首次构建必须有网络**，无法完全避免  
✅ **增量构建几乎不需要网络**（< 5 MB）  
💡 **推荐使用国内镜像源**，速度提升 5-10 倍

希望这个分析能帮助你规划构建环境！
