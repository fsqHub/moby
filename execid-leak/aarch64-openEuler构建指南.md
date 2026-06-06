# 在 aarch64 openEuler 上为 aarch64 CentOS 构建 Docker 包指南

## 目标

在 aarch64 openEuler 系统上构建 Docker v1.13.1 的 aarch64 版本，供 aarch64 CentOS 系统使用。提供两种交付形式：

1. **RPM 包**：适合生产环境，便于依赖管理和升级
2. **二进制包**：适合快速测试，直接替换 dockerd 即可

---

## 前置说明

### ✅ 完全可行

- Docker v1.13.1 **原生支持** aarch64 架构（存在 `Dockerfile.aarch64`）
- openEuler 和 CentOS 都基于 RPM 包管理，格式完全兼容
- 在 aarch64 openEuler 上构建的包可以直接用于 aarch64 CentOS

### 🎯 适用场景

- ARM64 服务器（华为鲲鹏、飞腾等）
- 树莓派集群
- aarch64 虚拟机环境

---

## 方案一：构建 RPM 包（推荐）✅

### 优势

- ✅ 自动处理依赖关系
- ✅ 便于在多台机器上部署
- ✅ 支持 yum/dnf 管理（升级、卸载）
- ✅ 包含 systemd 服务配置

### 操作步骤

#### 1.1 检查系统架构

```bash
# 确认当前系统架构
uname -m
# 预期输出: aarch64

# 确认操作系统
cat /etc/os-release | grep -E "NAME|VERSION"
# 预期输出: NAME="openEuler"
```

#### 1.2 安装构建依赖

```bash
# 更新系统
sudo yum update -y

# 安装基础构建工具
sudo yum install -y \
    git \
    gcc \
    make \
    rpm-build \
    rpmdevtools \
    createrepo \
    device-mapper-devel \
    btrfs-progs-devel \
    libseccomp-devel \
    pkgconfig \
    systemd-devel \
    tar \
    wget \
    which

# 安装 Docker（用于容器化构建）
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户加入 docker 组
sudo usermod -aG docker $USER
# 需要重新登录或执行: newgrp docker

# 验证 Docker 可用
docker version
docker info
```

#### 1.3 安装 Go 1.7.5 for ARM64

```bash
# 下载 Go 1.7.5 ARM64 版本
cd /tmp
wget https://golang.org/dl/go1.7.5.linux-arm64.tar.gz

# 如果下载失败，使用镜像源
# wget https://mirrors.ustc.edu.cn/golang/go1.7.5.linux-arm64.tar.gz

# 解压并安装
sudo tar -C /usr/local -xzf go1.7.5.linux-arm64.tar.gz

# 配置环境变量
cat >> ~/.bashrc <<'EOF'
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
EOF

source ~/.bashrc

# 验证安装
go version
# 预期输出: go version go1.7.5 linux/arm64
```

#### 1.4 准备源码

```bash
# 创建工作目录
mkdir -p ~/docker-patch-test
cd ~/docker-patch-test

# 克隆 moby 仓库
git clone https://github.com/moby/moby.git moby-1.13.1
cd moby-1.13.1

# 切换到 v1.13.1 标签
git checkout v1.13.1

# 创建工作分支
git checkout -b aarch64-build

# 验证 aarch64 支持
ls -la Dockerfile.aarch64
# 应该存在此文件
```

#### 1.5 适配 aarch64 RPM 构建配置

v1.13.1 原始代码只包含 amd64 的 RPM 构建配置，需要为 aarch64 创建：

```bash
cd ~/docker-patch-test/moby-1.13.1

# 检查现有配置
ls contrib/builder/rpm/
# 输出: amd64

# 创建 aarch64 配置目录
mkdir -p contrib/builder/rpm/aarch64

# 复制 amd64 配置作为模板
cp -r contrib/builder/rpm/amd64/* contrib/builder/rpm/aarch64/

# 查看需要修改的发行版
ls contrib/builder/rpm/aarch64/
# 输出: centos-7/ fedora-24/ fedora-25/ ...
```

**关键修改**：将构建容器的 FROM 镜像改为 ARM64 版本

```bash
# 以 centos-7 为例
cd contrib/builder/rpm/aarch64/centos-7

# 编辑 Dockerfile
vim Dockerfile

# 修改 FROM 行:
# 原始: FROM centos:7
# 修改为: FROM arm64v8/centos:7

# 保存退出后提交修改
cd ~/docker-patch-test/moby-1.13.1
git add contrib/builder/rpm/aarch64/
git commit -m "Add aarch64 RPM build config"
```

**批量修改脚本**（可选）:

```bash
cd ~/docker-patch-test/moby-1.13.1/contrib/builder/rpm/aarch64

# 批量替换所有 Dockerfile 的 FROM 镜像
find . -name "Dockerfile" -exec sed -i 's|FROM centos:|FROM arm64v8/centos:|g' {} \;
find . -name "Dockerfile" -exec sed -i 's|FROM fedora:|FROM arm64v8/fedora:|g' {} \;

# 验证修改
grep "^FROM" */Dockerfile
```

#### 1.6 构建 RPM 包

```bash
cd ~/docker-patch-test/moby-1.13.1

# 设置环境变量
export VERSION=1.13.1
export DOCKER_GITCOMMIT=$(git rev-parse --short HEAD)
export DOCKERFILE=Dockerfile.aarch64  # 使用 aarch64 构建文件
export DOCKER_BUILD_PKGS="centos-7"   # 只构建 CentOS 7 的包

# 步骤 1: 构建 Docker 开发环境镜像
make build

# 此步骤会:
# - 使用 Dockerfile.aarch64 构建开发容器
# - 下载并编译依赖（runc, containerd, libnetwork 等）
# - 预计耗时: 10-30 分钟（首次）

# 步骤 2: 编译动态链接的二进制（RPM 需要）
make dynbinary

# 此步骤会:
# - 在容器中编译 dockerd 和 docker-proxy
# - 生成 bundles/1.13.1/dynbinary-daemon/ 目录
# - 预计耗时: 5-10 分钟

# 验证二进制文件
ls -lh bundles/1.13.1/dynbinary-daemon/
# 应该看到: dockerd-1.13.1

file bundles/1.13.1/dynbinary-daemon/dockerd-1.13.1
# 预期输出: dockerd-1.13.1: ELF 64-bit LSB executable, ARM aarch64, ...

# 步骤 3: 构建 RPM 包
make rpm

# 此步骤会:
# - 为 CentOS 7 aarch64 构建 RPM 包
# - 包含 docker-engine 和 docker-engine-selinux
# - 预计耗时: 5-10 分钟
```

**构建过程说明**:

- `make build`: 创建包含所有构建工具的容器镜像
- `make dynbinary`: 编译动态链接的二进制（依赖系统库，适合打包）
- `make rpm`: 调用 `hack/make/build-rpm` 脚本，使用 rpmbuild 打包

**如果构建失败**，检查：

```bash
# 检查 Docker 是否运行
docker ps

# 检查磁盘空间（至少需要 10 GB）
df -h

# 查看构建日志
cat bundles/1.13.1/build-rpm/test.log

# 手动运行构建容器（调试用）
docker run -it --rm -v $(pwd):/go/src/github.com/docker/docker \
    docker:1.13.1-dev /bin/bash
```

#### 1.7 查找生成的 RPM 包

```bash
cd ~/docker-patch-test/moby-1.13.1

# 查找所有生成的 RPM 包
find bundles/1.13.1/ -name "*.rpm"

# 典型输出:
# bundles/1.13.1/build-rpm/centos-7/RPMS/aarch64/docker-engine-1.13.1-1.el7.centos.aarch64.rpm
# bundles/1.13.1/build-rpm/centos-7/RPMS/noarch/docker-engine-selinux-1.13.1-1.el7.centos.noarch.rpm
```

**验证 RPM 包**:

```bash
# 查看包信息
rpm -qip bundles/1.13.1/build-rpm/centos-7/RPMS/aarch64/docker-engine-*.rpm

# 确认架构
rpm -qp --queryformat '%{ARCH}\n' bundles/1.13.1/build-rpm/centos-7/RPMS/aarch64/docker-engine-*.rpm
# 输出: aarch64

# 查看包含的文件
rpm -qlp bundles/1.13.1/build-rpm/centos-7/RPMS/aarch64/docker-engine-*.rpm | head -20

# 预期包含:
# /usr/bin/dockerd
# /usr/bin/docker-proxy
# /usr/lib/systemd/system/docker.service
# /usr/lib/systemd/system/docker.socket
# ...
```

#### 1.8 在 aarch64 CentOS 上安装测试

将 RPM 包复制到 aarch64 CentOS 机器：

```bash
# 方法 1: scp 传输
scp bundles/1.13.1/build-rpm/centos-7/RPMS/aarch64/*.rpm root@centos-aarch64:/tmp/

# 方法 2: 创建本地 yum 仓库（多台机器部署）
mkdir -p ~/docker-rpms
cp bundles/1.13.1/build-rpm/centos-7/RPMS/*/*.rpm ~/docker-rpms/
createrepo ~/docker-rpms/
# 然后通过 HTTP 服务提供
```

**在 aarch64 CentOS 上安装**:

```bash
# 停止并卸载旧版本 Docker（如果有）
sudo systemctl stop docker
sudo yum remove -y docker docker-engine docker-ce

# 安装新编译的 RPM
sudo yum install -y /tmp/docker-engine-1.13.1-*.rpm
# 或者
sudo rpm -ivh /tmp/docker-engine-*.rpm

# 启动 Docker
sudo systemctl daemon-reload
sudo systemctl start docker
sudo systemctl enable docker

# 验证安装
docker version
docker info

# 运行测试容器
docker run --rm hello-world
```

**验证架构一致性**:

```bash
# 查看已安装的 Docker 架构
rpm -q docker-engine --queryformat '%{ARCH}\n'
# 输出: aarch64

# 查看 dockerd 二进制架构
file /usr/bin/dockerd
# 输出: /usr/bin/dockerd: ELF 64-bit LSB executable, ARM aarch64, ...
```

---

## 方案二：构建二进制包（快速测试）✅

### 适用场景

- 快速验证补丁效果
- 不需要完整的包管理
- 测试环境（非生产）

### 优势

- ✅ 构建速度更快（跳过 RPM 打包步骤）
- ✅ 直接替换二进制，无需卸载旧版本
- ✅ 适合快速迭代测试

### 操作步骤

#### 2.1 前置准备

前置步骤与方案一的 1.1-1.4 相同（安装依赖、Go、准备源码）。

#### 2.2 编译静态链接二进制

```bash
cd ~/docker-patch-test/moby-1.13.1

# 设置环境变量
export VERSION=1.13.1
export DOCKER_GITCOMMIT=$(git rev-parse --short HEAD)
export DOCKERFILE=Dockerfile.aarch64

# 方式 1: 编译静态链接二进制（推荐，无依赖）
make binary

# 此命令会:
# - 编译静态链接的 dockerd
# - 生成 bundles/1.13.1/binary-daemon/ 目录
# - 预计耗时: 5-10 分钟

# 查看生成的二进制
ls -lh bundles/1.13.1/binary-daemon/
# 输出: dockerd-1.13.1  docker-proxy-1.13.1
```

**验证二进制**:

```bash
# 检查架构
file bundles/1.13.1/binary-daemon/dockerd-1.13.1
# 输出: dockerd-1.13.1: ELF 64-bit LSB executable, ARM aarch64, statically linked, ...

# 检查大小（静态链接通常较大）
du -h bundles/1.13.1/binary-daemon/dockerd-1.13.1
# 预期: 30-50 MB

# 测试运行
bundles/1.13.1/binary-daemon/dockerd-1.13.1 --version
# 输出: Docker version 1.13.1, build ...
```

#### 2.3 打包二进制

```bash
cd ~/docker-patch-test/moby-1.13.1

# 创建二进制包目录
mkdir -p docker-1.13.1-aarch64-bin

# 复制主要二进制文件
cp bundles/1.13.1/binary-daemon/dockerd-1.13.1 docker-1.13.1-aarch64-bin/dockerd
cp bundles/1.13.1/binary-daemon/docker-proxy-1.13.1 docker-1.13.1-aarch64-bin/docker-proxy

# 复制辅助工具（如果需要）
# runc, containerd 等在 bundles/1.13.1/binary-daemon/ 或通过 vendor 获取

# 创建安装脚本
cat > docker-1.13.1-aarch64-bin/install.sh <<'EOF'
#!/bin/bash
set -e

echo "=== Docker 1.13.1 aarch64 安装脚本 ==="

# 检查权限
if [ "$EUID" -ne 0 ]; then
    echo "请使用 root 或 sudo 运行此脚本"
    exit 1
fi

# 停止现有 Docker 服务
systemctl stop docker || true

# 备份现有二进制
if [ -f /usr/bin/dockerd ]; then
    cp /usr/bin/dockerd /usr/bin/dockerd.bak.$(date +%Y%m%d%H%M%S)
    echo "已备份原 dockerd 到 /usr/bin/dockerd.bak.*"
fi

# 复制新二进制
cp -f dockerd /usr/bin/dockerd
cp -f docker-proxy /usr/bin/docker-proxy

# 设置权限
chmod +x /usr/bin/dockerd
chmod +x /usr/bin/docker-proxy

# 重启 Docker 服务
systemctl daemon-reload
systemctl start docker

# 验证版本
echo "=== 安装完成，验证版本 ==="
dockerd --version
docker version

echo "=== Docker 已成功更新到 1.13.1 ==="
EOF

chmod +x docker-1.13.1-aarch64-bin/install.sh

# 打包
tar -czf docker-1.13.1-aarch64-bin.tar.gz docker-1.13.1-aarch64-bin/

# 查看包大小
ls -lh docker-1.13.1-aarch64-bin.tar.gz
```

#### 2.4 在 aarch64 CentOS 上安装

```bash
# 传输二进制包到目标机器
scp docker-1.13.1-aarch64-bin.tar.gz root@centos-aarch64:/tmp/

# 在目标机器上
cd /tmp
tar -xzf docker-1.13.1-aarch64-bin.tar.gz
cd docker-1.13.1-aarch64-bin

# 运行安装脚本
sudo ./install.sh

# 或手动安装
sudo systemctl stop docker
sudo cp dockerd /usr/bin/dockerd
sudo cp docker-proxy /usr/bin/docker-proxy
sudo systemctl start docker

# 验证
docker version
docker info
```

**验证二进制是否正确运行**:

```bash
# 检查 dockerd 进程
ps aux | grep dockerd

# 查看二进制架构
file /usr/bin/dockerd
# 输出: /usr/bin/dockerd: ELF 64-bit LSB executable, ARM aarch64, ...

# 运行测试容器
docker run --rm arm64v8/alpine uname -m
# 输出: aarch64
```

---

## 逐步应用 Patches 的完整流程

### 3.1 为每个 Patch Group 构建独立版本

```bash
cd ~/docker-patch-test/moby-1.13.1

# 创建基线分支
git checkout -b baseline
git tag baseline-v1.13.1

# 应用 Patch Group 1
git checkout -b patch-group-1
git am ~/docker-patch-test/patches/0001-Fix-303111-dockerd-leaks-ExecIds-on-failed-exec-i.patch

# 构建 RPM 或二进制
export VERSION=1.13.1-patch1
make rpm  # 或 make binary

# 保存产物
mkdir -p ~/docker-builds/patch-group-1
cp bundles/*/build-rpm/centos-7/RPMS/aarch64/*.rpm ~/docker-builds/patch-group-1/
# 或
cp bundles/*/binary-daemon/dockerd-* ~/docker-builds/patch-group-1/

# 继续下一组...
git checkout -b patch-group-2 baseline-v1.13.1
git am patch2.patch patch3.patch
export VERSION=1.13.1-patch2
make rpm
# ...
```

### 3.2 版本标记建议

```bash
# 为每个构建版本添加自定义标记
export VERSION=1.13.1+patch1    # 基础修复
export VERSION=1.13.1+patch2    # + Stream 重构
export VERSION=1.13.1+patch3    # + I/O timeout
export VERSION=1.13.1+patch4    # + Context cancel
export VERSION=1.13.1+patch5    # + Client disconnect（完整修复）

# 在 RPM 包名中会体现为:
# docker-engine-1.13.1+patch1-1.el7.centos.aarch64.rpm
```

---

## 对比：RPM vs 二进制包

| 特性 | RPM 包 | 二进制包 |
|------|--------|----------|
| **构建时间** | 较长（15-20分钟） | 较短（5-10分钟） |
| **依赖管理** | ✅ 自动处理 | ❌ 需要手动确保 |
| **systemd 集成** | ✅ 自动配置 | ⚠️ 需要已有配置 |
| **升级便利性** | ✅ yum update | ❌ 手动替换 |
| **回滚** | ✅ yum downgrade | ⚠️ 需要备份 |
| **适用场景** | 生产环境 | 测试环境 |
| **快速迭代** | ❌ | ✅ |

---

## 常见问题排查

### 问题 1: Docker 容器无法启动（构建时）

```bash
# 检查 Docker 服务
sudo systemctl status docker

# 检查是否可以拉取镜像
docker pull arm64v8/centos:7

# 如果拉取失败，配置镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
sudo systemctl restart docker
```

### 问题 2: 编译失败 - "cannot find package"

```bash
# 检查 GOPATH
echo $GOPATH

# 清理并重新构建
make clean
rm -rf bundles/
make build
```

### 问题 3: RPM 构建失败 - "Dockerfile not found"

```bash
# 确认已设置正确的 DOCKERFILE 变量
export DOCKERFILE=Dockerfile.aarch64

# 验证文件存在
ls -la Dockerfile.aarch64

# 或者直接修改 Makefile
vim Makefile
# 找到 DOCKERFILE := ... 行，改为 Dockerfile.aarch64
```

### 问题 4: 安装 RPM 时依赖缺失

```bash
# 在目标 CentOS 机器上安装依赖
sudo yum install -y \
    device-mapper \
    iptables \
    libseccomp \
    systemd

# 如果仍然缺失，查看 RPM 的依赖列表
rpm -qpR docker-engine-*.rpm
```

### 问题 5: dockerd 启动失败

```bash
# 查看详细日志
sudo journalctl -xeu docker.service

# 手动运行 dockerd 查看错误
sudo /usr/bin/dockerd -D

# 常见原因:
# 1. SELinux 阻止 - sudo setenforce 0
# 2. 缺少 /var/lib/docker 目录 - sudo mkdir -p /var/lib/docker
# 3. iptables 未安装 - sudo yum install iptables
```

---

## 总结

### 推荐工作流

1. **首次构建**：使用方案一（RPM）验证完整构建流程
2. **快速迭代**：使用方案二（二进制）快速测试各 patch group
3. **最终交付**：使用方案一（RPM）打包确定的修复版本

### 预期时间

- **环境准备**：30-60 分钟（首次）
- **RPM 包构建**：15-20 分钟/次
- **二进制构建**：5-10 分钟/次
- **5 组 patches 完整验证**：2-3 小时

### 核心优势

✅ 在 aarch64 openEuler 上构建 aarch64 CentOS 包**完全可行**  
✅ 无需交叉编译，无架构兼容性问题  
✅ 二进制包方式可以极大加快验证速度  
✅ Docker v1.13.1 原生支持 ARM64，无需额外适配

**开始构建吧！** 有问题随时参考本文档的故障排查章节。
