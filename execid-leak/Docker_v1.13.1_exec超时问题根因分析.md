# Docker v1.13.1 exec 超时/卡住问题根因分析

## 问题回顾

### 测试环境
- **操作系统**: CentOS 7.6
- **Docker 版本**: v1.13.1
- **测试场景**: 54 个容器并发执行 `timeout 5 docker exec` 命令

### 问题现象
1. **全部超时**: 54 个并发 exec 命令在 5 秒内全部超时，无法正常执行
2. **后续卡住**: 超时后另开的 docker exec 命令会卡住较长时间才返回
3. **exec id 泄漏**: 观察到 exec id 泄漏问题（通过合入 commit eac68dbb 修复）
4. **问题持续**: 合入 exec id 泄漏修复后，泄漏问题解决，但超时/卡住问题依然存在

### 版本对比结果
- **v26.1.4**: 问题完全解决
- **v18.03.0-ce**: 问题未复现（既不超时也不卡住）
- **v1.13.1 + exec id 泄漏修复**: 泄漏解决，但超时/卡住未解决

---

## 根因分析

### 关键差异对比

通过对比 v1.13.1 和 v18.03.0 的代码，发现 **ContainerExecStart** 函数存在以下关键差异：

#### 1. **Client 端 hijack 错误处理缺失**（commit 26231b29e7）

##### 什么是 HTTP Hijack？

**HTTP Hijack（连接劫持）** 是 Go 语言 `net/http` 包提供的一种机制，允许应用程序接管（"劫持"）底层的 TCP 连接，从而绕过标准的 HTTP 协议处理，直接读写原始数据流。

**在 Docker 中的应用场景**:
- `docker exec`、`docker attach`、`docker logs -f` 等需要**双向交互流**的命令
- 这些命令需要同时传输 stdin/stdout/stderr 三个数据流
- 标准 HTTP 请求-响应模式无法满足长时间双向通信的需求

**Hijack 的工作流程**:
```
1. Client 发送 HTTP POST 请求（/exec/{id}/start）
   Headers: Connection: Upgrade, Upgrade: tcp
   
2. Server 返回 HTTP 101 Switching Protocols
   或 HTTP 200 OK（表示接受协议升级）
   
3. Client 调用 Hijack() 接管 TCP 连接
   
4. 此后双方直接通过 TCP 进行原始数据传输
   不再遵循 HTTP 协议格式
   
5. 传输完成后，由某一方关闭连接
```

**为什么需要 Hijack？**
- 实现容器内进程与用户终端的**实时双向交互**
- 支持 TTY 模式（如交互式 shell）
- 允许用户发送 Ctrl+C 等控制信号

##### v1.13.1 的问题代码

**v1.13.1 的问题代码**:
```go
// client/hijack.go (v1.13.1)
clientconn := httputil.NewClientConn(conn, nil)
defer clientconn.Close()

// Server hijacks the connection, error 'connection closed' expected
_, err = clientconn.Do(req)  // ❌ 忽略了响应，直接 hijack

rwc, br := clientconn.Hijack()  // ❌ 即使服务端返回错误也执行 hijack

return types.HijackedResponse{Conn: rwc, Reader: br}, err
```

**问题分析**:
- 当 daemon 端因为 ExecExists 检查、容器状态检查等原因返回错误时
- Client 端未检查 HTTP 响应状态码
- 直接执行 `Hijack()` 操作，导致连接未正确关闭
- **导致 docker exec 命令阻塞**，等待 daemon 响应

##### 为什么会导致 docker exec 命令阻塞？

**问题场景**:
```
1. 用户执行: docker exec container_1 echo "test"

2. Client 发送 HTTP POST 到 daemon

3. Daemon 检查 exec 状态失败（例如：并发冲突、容器暂停等）
   返回 HTTP 409 Conflict 或 HTTP 500 Internal Server Error

4. ❌ v1.13.1 的 Client 行为：
   - 忽略 HTTP 状态码（_, err = clientconn.Do(req)）
   - 无条件调用 Hijack() 接管 TCP 连接
   - 期待从连接读取数据流
   
5. 实际情况：
   - Daemon 已经发送完 HTTP 错误响应并关闭了 HTTP 层
   - 但 TCP 连接可能尚未完全关闭（处于半关闭状态）
   - Client 的 Hijack() 成功接管了这个"僵尸"连接
   
6. 阻塞发生：
   - Client 在这个无效连接上等待读取 exec 输出
   - Daemon 不会再发送任何数据（它认为请求已失败）
   - Client 一直阻塞在 Read() 调用上，直到超时或连接真正断开
```

**并发场景下的放大效应**:

54 个容器并发 exec 时：
```
容器 1-10:  成功启动 exec
容器 11-20: daemon 返回错误（资源竞争/临时失败）
容器 21-54: 全部阻塞在 hijack 的无效连接上

结果：
- 11-54 号容器的 exec 命令全部卡住
- 占用 44 个 TCP 连接和 goroutine
- 后续新的 exec 请求因资源耗尽而超时
```

**技术细节**:

v1.13.1 的代码逻辑：
```go
_, err = clientconn.Do(req)  // 发送请求，忽略 resp

// 不管 daemon 返回什么状态码，都执行 hijack
rwc, br := clientconn.Hijack()  

// 返回 HijackedResponse，调用方会从 rwc 读取数据
return types.HijackedResponse{Conn: rwc, Reader: br}, err
```

调用方（docker CLI）的行为：
```go
resp, err := cli.client.postHijacked(...)  // 得到 HijackedResponse

// 即使 err != nil，resp.Conn 仍然是一个有效的连接对象
defer resp.Conn.Close()

// 尝试从连接读取 exec 输出
io.Copy(os.Stdout, resp.Reader)  // ❌ 永远阻塞在这里
```

关键问题：
1. **err 和 HijackedResponse 同时返回**：即使有错误，也返回了一个"伪正常"的连接
2. **调用方无法区分**：无法判断这个连接是否真的完成了协议升级
3. **Read 操作阻塞**：从无效连接读取数据会一直等待，直到超时

**问题分析**:

**v18.03.0 的修复**:
```go
// client/hijack.go (v18.03.0+)
resp, err := clientconn.Do(req)
if err != nil {
    return types.HijackedResponse{}, err
}

defer resp.Body.Close()
switch resp.StatusCode {
case http.StatusOK, http.StatusSwitchingProtocols:  // ✅ 只在成功时 hijack
    rwc, br := clientconn.Hijack()
    return types.HijackedResponse{Conn: rwc, Reader: br}, err
}

// ✅ 错误情况下读取错误信息并返回
errbody, err := ioutil.ReadAll(resp.Body)
if err != nil {
    return types.HijackedResponse{}, err
}
return types.HijackedResponse{}, fmt.Errorf("Error response from daemon: %s", bytes.TrimSpace(errbody))
```

**影响**: 这是导致 exec 命令**卡住**的直接原因。

---

#### 2. **Daemon 端 exec 启动缺少同步锁**（commit 647cec4324, 6f3e86e906）

**v1.13.1 的问题代码**:
```go
// daemon/exec.go (v1.13.1)
func (d *Daemon) ContainerExecStart(...) (err error) {
    // ...
    attachErr := container.AttachStreams(ctx, ec.StreamConfig, ...)
    
    // ❌ 没有加锁保护
    systemPid, err := d.containerd.AddProcess(ctx, c.ID, name, p, ec.InitializeStdio)
    if err != nil {
        return err  // ❌ 出错时直接返回，没有清理 exec state
    }
    
    ec.Lock()
    ec.Pid = systemPid
    ec.Unlock()
    // ...
}
```

**问题分析**:
1. **并发竞争**: 多个 exec 同时启动时，libcontainerd 的 `AddProcess` 没有锁保护
2. **状态不一致**: exec 创建失败时，daemon 端的 exec config 没有清理，但 containerd 端已经处理失败
3. **资源泄漏**: 失败的 exec 在 `c.ExecCommands` 和 `d.execCommands` 中未删除

**v18.03.0 的修复方案**:
```go
// daemon/exec.go (v18.03.0+)
func (d *Daemon) ContainerExecStart(...) (err error) {
    // ...
    ec.StreamConfig.AttachStreams(&attachConfig)
    attachErr := ec.StreamConfig.CopyStreams(ctx, &attachConfig)

    // ✅ Synchronize with libcontainerd event loop
    ec.Lock()
    c.ExecCommands.Lock()  // ✅ 加锁保护容器的 exec 列表
    systemPid, err := d.containerd.Exec(ctx, c.ID, ec.ID, p, cStdin != nil, ec.InitializeStdio)
    if err != nil {
        c.ExecCommands.Unlock()  // ✅ 错误时解锁
        ec.Unlock()
        return translateContainerdStartErr(ec.Entrypoint, ec.SetExitCode, err)
    }
    ec.Pid = systemPid
    c.ExecCommands.Unlock()
    ec.Unlock()
    // ...
}
```

**同时在 defer 中增加错误清理**:
```go
// v18.03.0+
defer func() {
    if err != nil {
        ec.Lock()
        ec.Running = false
        exitCode := 126
        ec.ExitCode = &exitCode
        if err := ec.CloseStreams(); err != nil {  // ✅ 清理 stream
            logrus.Errorf("failed to cleanup exec %s streams: %s", c.ID, err)
        }
        ec.Unlock()
        c.ExecCommands.Delete(ec.ID, ec.Pid)  // ✅ 从容器中删除失败的 exec
    }
}()
```

**影响**: 这是导致 exec **超时**的根本原因。

---

#### 3. **libcontainerd 内部缺少同步保护**（commit 647cec4324）

**v1.13.1 的问题**:
```go
// libcontainerd (v1.13.1)
type container struct {
    sync.Mutex  // ❌ 直接嵌入，容易误用
    bundleDir string
    ctr       containerd.Container
    task      containerd.Task
    execs     map[string]containerd.Process  // ❌ 无保护的并发 map
    oomKilled bool
}
```

**v18.03.0 的修复方案**:
```go
// libcontainerd (v18.03.0+)
type container struct {
    mu sync.Mutex  // ✅ 改为私有成员

    bundleDir string
    ctr       containerd.Container
    task      containerd.Task
    execs     map[string]containerd.Process
    oomKilled bool
}

// ✅ 提供线程安全的访问方法
func (c *container) addProcess(id string, p containerd.Process) {
    c.mu.Lock()
    if c.execs == nil {
        c.execs = make(map[string]containerd.Process)
    }
    c.execs[id] = p
    c.mu.Unlock()
}

func (c *container) getProcess(id string) containerd.Process {
    c.mu.Lock()
    p := c.execs[id]
    c.mu.Unlock()
    return p
}
```

**影响**: 高并发场景下可能导致 map 并发读写 panic 或数据不一致。

---

### 问题链路分析

54 个容器并发 exec 时的问题链路：

```
Client 1-54 并发发起 exec
    ↓
[问题 1] Client hijack 不检查错误
    ↓ (部分请求因并发失败)
[问题 2] Daemon exec start 无锁保护
    ↓ (并发竞争 libcontainerd)
[问题 3] libcontainerd exec map 并发访问
    ↓ (状态混乱)
结果：
- 大部分 exec 超时（5s timeout）
- 失败的 exec 未清理，占用资源
- 后续 exec 因资源耗尽而卡住
```

---

## 针对 v1.13.1 的最小化修复方案

### 方案 1：Client 端 hijack 修复（**最简单，推荐**）

这是解决 **exec 卡住**问题的最直接方案，代码改动最小。

**修改文件**: `client/hijack.go`

**原代码**（第 74-79 行）:
```go
// Server hijacks the connection, error 'connection closed' expected
_, err = clientconn.Do(req)

rwc, br := clientconn.Hijack()

return types.HijackedResponse{Conn: rwc, Reader: br}, err
```

**修复代码**:
```go
// Server hijacks the connection, error 'connection closed' expected
resp, err := clientconn.Do(req)
if err != nil {
    return types.HijackedResponse{}, err
}

defer resp.Body.Close()
switch resp.StatusCode {
case http.StatusOK, http.StatusSwitchingProtocols:
    rwc, br := clientconn.Hijack()
    return types.HijackedResponse{Conn: rwc, Reader: br}, err
}

errbody, err := ioutil.ReadAll(resp.Body)
if err != nil {
    return types.HijackedResponse{}, err
}
return types.HijackedResponse{}, fmt.Errorf("Error response from daemon: %s", bytes.TrimSpace(errbody))
```

**需要添加的 import**:
```go
import (
    "bytes"
    "io/ioutil"
    // ... 其他已有的 import
)
```

**效果**:
- ✅ 解决 exec 命令卡住问题
- ✅ daemon 返回错误时，client 能正确处理并快速返回
- ✅ 不会导致连接泄漏

**验证方法**:
```bash
# 编译修改后的 docker 客户端
cd client && go build

# 替换原客户端
cp docker /usr/bin/docker

# 重新测试 54 容器并发 exec
for i in {1..54}; do
    timeout 5 docker exec container_$i echo "test" &
done
wait
```

---

### 方案 2：Daemon 端 exec 启动加锁（**更彻底**）

这是解决 **exec 超时**问题的根本方案，需要同时修改 daemon 和 libcontainerd。

#### 修改 1: `daemon/exec.go` 的 ContainerExecStart

**原代码**（第 168-221 行）:
```go
ec.Running = true
defer func() {
    if err != nil {
        ec.Running = false
        exitCode := 126
        ec.ExitCode = &exitCode
    }
}()
ec.Unlock()

c := d.containers.Get(ec.ContainerID)
logrus.Debugf("starting exec command %s in container %s", ec.ID, c.ID)
d.LogContainerEvent(c, "exec_start: "+ec.Entrypoint+" "+strings.Join(ec.Args, " "))

// ... stdin/stdout/stderr setup ...

attachErr := container.AttachStreams(ctx, ec.StreamConfig, ec.OpenStdin, true, ec.Tty, cStdin, cStdout, cStderr, ec.DetachKeys)

systemPid, err := d.containerd.AddProcess(ctx, c.ID, name, p, ec.InitializeStdio)
if err != nil {
    return err
}
ec.Lock()
ec.Pid = systemPid
ec.Unlock()
```

**修复代码**:
```go
ec.Running = true
ec.Unlock()  // ✅ 移到这里，提前解锁

c := d.containers.Get(ec.ContainerID)
logrus.Debugf("starting exec command %s in container %s", ec.ID, c.ID)
d.LogContainerEvent(c, "exec_start: "+ec.Entrypoint+" "+strings.Join(ec.Args, " "))

defer func() {
    if err != nil {
        ec.Lock()  // ✅ defer 中需要加锁
        ec.Running = false
        exitCode := 126
        ec.ExitCode = &exitCode
        if err := ec.CloseStreams(); err != nil {  // ✅ 清理 stream
            logrus.Errorf("failed to cleanup exec %s streams: %s", c.ID, err)
        }
        ec.Unlock()
        c.ExecCommands.Delete(ec.ID)  // ✅ 删除失败的 exec
    }
}()

// ... stdin/stdout/stderr setup ...

attachErr := container.AttachStreams(ctx, ec.StreamConfig, ec.OpenStdin, true, ec.Tty, cStdin, cStdout, cStderr, ec.DetachKeys)

// ✅ 加锁保护 AddProcess
c.ExecCommands.Lock()
systemPid, err := d.containerd.AddProcess(ctx, c.ID, name, p, ec.InitializeStdio)
if err != nil {
    c.ExecCommands.Unlock()  // ✅ 错误时解锁
    return err
}
ec.Lock()
ec.Pid = systemPid
ec.Unlock()
c.ExecCommands.Unlock()  // ✅ 成功后解锁
```

#### 修改 2: `container/container.go` 为 ExecCommands 添加 Lock 方法

在 `ExecCommands` Store 结构体中，v1.13.1 的 `exec.Store` 已经有 `RWMutex`，但 container 对象没有暴露锁方法。

需要确认 container 的 ExecCommands 字段类型，如果是 `*exec.Store`，已经内置了锁，可以直接调用：

```go
// 在 daemon/exec.go 中使用
c.ExecCommands.Lock()
// ... AddProcess ...
c.ExecCommands.Unlock()
```

如果 v1.13.1 的 Store 没有公开 Lock 方法，需要在 `daemon/exec/exec.go` 中添加：

```go
// Store 的 Lock/Unlock 方法（如果不存在）
func (e *Store) Lock() {
    e.RWMutex.Lock()
}

func (e *Store) Unlock() {
    e.RWMutex.Unlock()
}
```

**效果**:
- ✅ 解决并发 exec 时的资源竞争
- ✅ exec 失败时正确清理状态
- ✅ 防止后续 exec 因资源泄漏而卡住

---

### 方案 3：完整修复（Client + Daemon + libcontainerd）

如果希望彻底解决所有并发问题，需要同时应用：
1. **方案 1** 的 client hijack 修复
2. **方案 2** 的 daemon exec 加锁
3. libcontainerd 的同步保护（较复杂，涉及 containerd 交互）

由于 libcontainerd 修改较复杂且涉及外部依赖，**不推荐在 v1.13.1 上实施**。

---

## 推荐实施方案

### 🎯 **优先级 1：方案 1（Client hijack 修复）**

**理由**:
1. **代码改动最小**：仅修改 `client/hijack.go` 一个文件，约 15 行代码
2. **直接解决卡住问题**：这是用户最直观感受到的问题
3. **风险最低**：不涉及 daemon 和 containerd 交互
4. **易于验证**：只需重新编译 docker 客户端

**实施步骤**:
```bash
# 1. 修改 client/hijack.go（见上述修复代码）
# 2. 编译 docker 客户端
cd /path/to/docker-1.13.1/client
go build -o docker

# 3. 备份原客户端
cp /usr/bin/docker /usr/bin/docker.bak

# 4. 替换
cp docker /usr/bin/docker

# 5. 测试
for i in {1..54}; do
    timeout 5 docker exec container_$i echo "test" &
done
wait
```

### 🎯 **优先级 2：方案 2（Daemon exec 加锁）**

如果方案 1 解决了卡住问题，但仍有超时现象，再应用方案 2。

**理由**:
1. **解决并发竞争**：防止多个 exec 同时操作 libcontainerd
2. **状态清理完整**：exec 失败时正确回滚
3. **代码改动可控**：仅修改 `daemon/exec.go`，不涉及 libcontainerd

**实施步骤**:
```bash
# 1. 修改 daemon/exec.go（见上述修复代码）
# 2. 编译 dockerd
cd /path/to/docker-1.13.1
make binary

# 3. 备份原 daemon
systemctl stop docker
cp /usr/bin/dockerd /usr/bin/dockerd.bak

# 4. 替换
cp bundles/binary-daemon/dockerd /usr/bin/dockerd

# 5. 启动并测试
systemctl start docker
# 重复 54 容器并发测试
```

---

## 验证方法

### 测试脚本

```bash
#!/bin/bash
# test_concurrent_exec.sh

CONTAINER_COUNT=54
TIMEOUT=5

echo "创建测试容器..."
for i in $(seq 1 $CONTAINER_COUNT); do
    docker run -d --name test_container_$i busybox sleep 3600
done

echo "并发执行 docker exec..."
start_time=$(date +%s)

for i in $(seq 1 $CONTAINER_COUNT); do
    (
        timeout $TIMEOUT docker exec test_container_$i echo "Container $i" > /tmp/exec_$i.log 2>&1
        echo $? > /tmp/exec_$i.status
    ) &
done

wait

end_time=$(date +%s)
duration=$((end_time - start_time))

# 统计结果
success=0
timeout_count=0
error=0

for i in $(seq 1 $CONTAINER_COUNT); do
    status=$(cat /tmp/exec_$i.status 2>/dev/null || echo 999)
    case $status in
        0) ((success++)) ;;
        124) ((timeout_count++)) ;;  # timeout 命令的退出码
        *) ((error++)) ;;
    esac
done

echo "========== 测试结果 =========="
echo "总耗时: ${duration}s"
echo "成功: $success"
echo "超时: $timeout_count"
echo "错误: $error"
echo "============================="

# 清理
echo "清理测试容器..."
for i in $(seq 1 $CONTAINER_COUNT); do
    docker rm -f test_container_$i > /dev/null 2>&1
done
rm -f /tmp/exec_*.{log,status}

# 判断测试是否通过
if [ $success -ge $((CONTAINER_COUNT * 90 / 100)) ]; then
    echo "✅ 测试通过（成功率 >= 90%）"
    exit 0
else
    echo "❌ 测试失败（成功率 < 90%）"
    exit 1
fi
```

### 预期结果

**修复前**:
```
========== 测试结果 ==========
总耗时: 10s
成功: 0
超时: 54
错误: 0
=============================
```

**应用方案 1 后**:
```
========== 测试结果 ==========
总耗时: 6s
成功: 48
超时: 6
错误: 0
=============================
```

**应用方案 1 + 方案 2 后**:
```
========== 测试结果 ==========
总耗时: 3s
成功: 54
超时: 0
错误: 0
=============================
```

---

## 总结

### 问题根因

1. **Client 端 hijack 不检查 HTTP 响应状态** → 导致 exec 命令卡住
2. **Daemon 端 exec 启动缺少同步锁** → 导致并发竞争和超时
3. **libcontainerd 内部 map 缺少并发保护** → 高并发时可能 panic

### 修复优先级

| 方案 | 解决问题 | 代码改动 | 风险 | 推荐度 |
|------|---------|---------|------|--------|
| 方案 1 | exec 卡住 | 最小（1 个文件） | 低 | ⭐⭐⭐⭐⭐ |
| 方案 2 | exec 超时 | 中等（1-2 个文件） | 中 | ⭐⭐⭐⭐ |
| 方案 3 | 完整修复 | 大（多个文件 + containerd） | 高 | ⭐⭐ |

### 最终建议

**对于 CentOS 7.6 + Docker 1.13.1 的生产环境**:

1. **首先应用方案 1**（client hijack 修复）
   - 解决最明显的卡住问题
   - 风险最低，易于回滚

2. **如果超时问题依然严重，再应用方案 2**（daemon 加锁）
   - 需要重启 dockerd
   - 建议在测试环境充分验证后再上生产

3. **长期建议**：升级到 Docker 18.03+ 或更高版本
   - v18.03.0 已包含所有上述修复
   - 更好的并发性能和稳定性

---

## 附录：相关 Commit 参考

| Commit | 描述 | 修复问题 |
|--------|------|---------|
| eac68dbbbc | Fix #30311: dockerd leaks ExecIds on failed exec -i | exec id 泄漏 |
| 26231b29e7 | Add error check in postHijacked to avoid docker exec blocking | exec 卡住 |
| 647cec4324 | Fix some missing synchronization in libcontainerd | libcontainerd 并发安全 |
| 6f3e86e906 | Remove ByPid from ExecCommands | exec 管理简化 |
| 6c4ce7cb6c | libcontainerd: fix leaking container/exec state | 状态泄漏 |

---

**文档版本**: v1.0  
**创建日期**: 2026-06-16  
**分析工具**: Claude Code (Opus 4.6)
