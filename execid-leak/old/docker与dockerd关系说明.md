# Docker 和 Dockerd 的关系详解

## 简单回答

- **`docker`**：Docker **客户端**（CLI），用户直接使用的命令行工具
- **`dockerd`**：Docker **守护进程**（Daemon），后台运行的服务端程序

它们是**客户端-服务器（Client-Server）架构**，通过 REST API 通信。

---

## 详细解释

### 1. dockerd - Docker Daemon（服务端）

**角色**：Docker 的核心引擎，负责所有实际工作

**职责**：
- ✅ 管理容器生命周期（创建、启动、停止、删除）
- ✅ 管理镜像（构建、拉取、推送）
- ✅ 管理网络和存储卷
- ✅ 与 containerd/runc 交互，操作底层容器
- ✅ 提供 REST API 供客户端调用
- ✅ 监听 Unix Socket 或 TCP 端口

**运行方式**：
```bash
# 作为 systemd 服务运行（常见）
sudo systemctl start docker
# 实际执行: /usr/bin/dockerd

# 查看 dockerd 进程
ps aux | grep dockerd
# 输出: /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

**监听位置**：
```bash
# 默认监听 Unix Socket
/var/run/docker.sock

# 也可以配置 TCP 监听
dockerd -H tcp://0.0.0.0:2375
```

**源码位置**（v1.13.1）：
```
moby/cmd/dockerd/
├── daemon.go           # 主入口
├── service.go          # 服务管理
└── ...
```

---

### 2. docker - Docker Client（客户端）

**角色**：用户交互的命令行工具

**职责**：
- ✅ 解析用户命令（如 `docker run`、`docker ps`）
- ✅ 将命令转换为 HTTP REST API 请求
- ✅ 发送请求到 dockerd（通过 Unix Socket 或 TCP）
- ✅ 接收 dockerd 返回的结果
- ✅ 格式化输出给用户

**运行方式**：
```bash
# 用户每次执行 docker 命令时运行
docker ps
docker run ubuntu echo hello
docker build -t myapp .
```

**源码位置**（v1.13.1）：
```
moby/cmd/docker/
├── docker.go           # 主入口
└── ...
```

**注意**：从 Docker 1.11+ 开始，客户端代码已经拆分到独立仓库：
- https://github.com/docker/cli （新版本）
- 但 v1.13.1 仍然在同一个 moby 仓库

---

## 通信机制

### 架构图

```
┌─────────────────────────────────────────────────────┐
│                       用户                           │
└─────────────────────────────────────────────────────┘
                         │
                         │ 执行命令
                         ▼
┌─────────────────────────────────────────────────────┐
│  docker (客户端)                                     │
│  - 解析命令                                          │
│  - 转换为 HTTP 请求                                  │
└─────────────────────────────────────────────────────┘
                         │
                         │ HTTP REST API
                         │ (通过 /var/run/docker.sock)
                         ▼
┌─────────────────────────────────────────────────────┐
│  dockerd (服务端)                                    │
│  - 监听 API 请求                                     │
│  - 执行实际操作                                      │
│  - 调用 containerd/runc                              │
└─────────────────────────────────────────────────────┘
                         │
                         │ gRPC
                         ▼
┌─────────────────────────────────────────────────────┐
│  containerd                                          │
│  - 容器运行时管理                                    │
└─────────────────────────────────────────────────────┘
                         │
                         │ exec
                         ▼
┌─────────────────────────────────────────────────────┐
│  runc                                                │
│  - 创建和运行容器                                    │
└─────────────────────────────────────────────────────┘
```

### 实际通信示例

当你执行 `docker ps` 时：

```bash
# 步骤 1: docker 客户端发送 HTTP 请求
docker ps
↓
# 步骤 2: 转换为 REST API 调用
GET /containers/json HTTP/1.1
Host: /var/run/docker.sock

# 步骤 3: dockerd 接收并处理
dockerd 收到请求 → 查询容器列表 → 返回 JSON

# 步骤 4: docker 客户端格式化输出
CONTAINER ID   IMAGE     COMMAND   ...
abc123def456   ubuntu    "bash"    ...
```

---

## 为什么要分离客户端和服务端？

### 1. 架构灵活性

**远程管理**：
```bash
# 客户端可以连接到远程 dockerd
docker -H tcp://192.168.1.100:2375 ps

# 甚至跨主机管理多个 Docker 环境
export DOCKER_HOST=tcp://server1:2375
docker ps

export DOCKER_HOST=tcp://server2:2375
docker ps
```

### 2. 权限隔离

- **dockerd**：需要 root 权限（操作 namespace、cgroup）
- **docker**：普通用户可以运行（通过 docker 组访问 socket）

```bash
# dockerd 必须以 root 运行
sudo systemctl status docker
# User: root

# docker 客户端可以是普通用户（通过 socket 权限）
ls -l /var/run/docker.sock
# srw-rw---- 1 root docker 0 ... /var/run/docker.sock

# 加入 docker 组的用户可以使用
groups
# fsq docker
```

### 3. 多种客户端

除了官方 `docker` 命令，还可以用其他客户端：

- **Docker Compose**：使用 Docker API
- **Portainer**：Web UI，调用 Docker API
- **自定义脚本**：直接调用 Docker REST API

```bash
# 直接调用 API（不使用 docker 命令）
curl --unix-socket /var/run/docker.sock http://localhost/containers/json
```

---

## v1.13.1 的构建产物

### 使用 `make binary` 构建

```bash
cd moby-1.13.1
make binary

# 生成的二进制文件：
ls bundles/1.13.1/binary-client/
# docker-1.13.1         ← 客户端

ls bundles/1.13.1/binary-daemon/
# dockerd-1.13.1        ← 服务端
# docker-proxy-1.13.1   ← 网络代理（容器端口转发）
```

### 使用 `make rpm` 构建

RPM 包会将两者都打包进去：

```bash
rpm -qlp docker-engine-1.13.1-*.rpm | grep bin
# /usr/bin/docker          ← 客户端
# /usr/bin/dockerd         ← 服务端
# /usr/bin/docker-proxy    ← 网络代理
# /usr/bin/docker-containerd
# /usr/bin/docker-containerd-shim
# /usr/bin/docker-runc
```

---

## 验证 Patches 时需要替换哪个？

### 关键回答：**只需要替换 `dockerd`**

ExecID 泄漏问题发生在**服务端（dockerd）**，因为：

- `docker exec` 命令由客户端发起，但实际执行在服务端
- ExecID 的管理（创建、存储、清理）在 `dockerd` 中
- 补丁修改的是 `daemon/exec.go`、`daemon/monitor.go` 等服务端代码

### 测试流程

```bash
# 方式 1: 只替换 dockerd（推荐）
sudo systemctl stop docker
sudo cp dockerd-1.13.1-patch1 /usr/bin/dockerd
sudo systemctl start docker

# 客户端 docker 不需要更新
docker version
# Client: 1.13.1 (旧版本，没关系)
# Server: 1.13.1 (新版本，包含补丁)

# 方式 2: 完整替换（可选）
sudo cp docker-1.13.1 /usr/bin/docker
sudo cp dockerd-1.13.1 /usr/bin/dockerd
sudo systemctl restart docker
```

### 验证补丁效果

```bash
# 运行复现脚本
./reproduce-issue.sh

# 检查 ExecID 泄漏
docker inspect test-1 | jq '.[].ExecIDs'

# 如果为空或只有运行中的 exec，说明补丁生效
```

---

## 常见误区

### ❌ 误区 1: 以为 docker 命令直接操作容器

**错误理解**：
```
docker run → 直接创建容器
```

**正确理解**：
```
docker run → 发送请求到 dockerd → dockerd 调用 containerd → containerd 调用 runc → 容器创建
```

### ❌ 误区 2: 以为更新 docker 客户端就能修复服务端 bug

**场景**：ExecID 泄漏问题

- ❌ 只更新 `/usr/bin/docker` → **无效**
- ✅ 更新 `/usr/bin/dockerd` → **有效**

### ❌ 误区 3: 以为 dockerd 死了就不能用 docker 命令

**现象**：
```bash
sudo systemctl stop docker

docker ps
# Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
# Is the docker daemon running?
```

**原因**：客户端能运行，但无法连接到服务端，所以报错。

---

## 快速检查命令

```bash
# 1. 查看二进制文件信息
file /usr/bin/docker
file /usr/bin/dockerd

# 2. 查看版本
docker version
# Client: ... ← docker 客户端版本
# Server: ... ← dockerd 服务端版本

# 3. 查看 dockerd 进程
ps aux | grep dockerd
# root  1234  ... /usr/bin/dockerd -H fd:// ...

# 4. 查看 API 通信
strace -e trace=connect docker ps 2>&1 | grep docker.sock
# connect(..., "/var/run/docker.sock", ...) = 0

# 5. 直接调用 API（绕过 docker 客户端）
curl --unix-socket /var/run/docker.sock http://localhost/version
```

---

## 总结

### 核心关系

```
docker (客户端)
    ↓ REST API (Unix Socket)
dockerd (服务端)
    ↓ gRPC
containerd
    ↓ exec
runc
```

### 角色分工

| 组件 | 角色 | 职责 | 用户接触 |
|------|------|------|----------|
| `docker` | 客户端 | 命令行界面，转换用户命令为 API 调用 | ✅ 直接使用 |
| `dockerd` | 服务端 | 容器管理，镜像管理，API 服务 | ❌ 后台运行 |
| `containerd` | 运行时 | 容器生命周期管理 | ❌ 被 dockerd 调用 |
| `runc` | 执行器 | 创建和运行容器进程 | ❌ 被 containerd 调用 |

### 补丁验证重点

**只需要关注 `dockerd`**：
- ✅ ExecID 泄漏问题在服务端
- ✅ 补丁修改的是 `daemon/` 目录代码
- ✅ 只需要替换 `/usr/bin/dockerd` 即可测试

**客户端（docker）可以保持不变**：
- ✅ API 接口向后兼容
- ✅ 客户端只是请求转发
- ✅ 不影响补丁验证

---

希望这个解释清楚了！简单说就是：**`docker` 是你用的工具，`dockerd` 是干活的服务**。验证补丁时，只需要替换 `dockerd`。
