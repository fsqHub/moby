# Patches 说明

本目录包含了从 v1.13.1 到 v26.1.4 期间解决 ExecID 泄漏和拓展场景问题的关键 commits 的 patch 文件。

## Patch 列表（按时间顺序）

### 1. 原始问题修复（2017-01）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-Fix-303111-dockerd-leaks-ExecIds-on-failed-exec-i.patch` | `3cc0d6bb04` | 2017-01-21 | 修复 #30311：exec 启动失败时删除 ExecCommands |

**首次出现版本**: v17.04.0-ce  
**来源仓库**: `/home/fsq/Desktop/fsq/moby`

---

### 2. Stream 重构与竞态修复（2017-01～2017-02）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-Move-attach-code-to-stream-package.patch` | `2ddec97545` | 2017-01-19 | 将 attach 代码重构到 stream 包 |
| `0001-Resolve-race-conditions-in-attach-API-call.patch` | `84d6240cfe` | 2017-01-30 | 修复 attach API 竞态条件 |

**首次出现版本**: v17.04.0-ce  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

---

### 3. Runtime 与 I/O 处理改进（2019-06）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-Handle-blocked-I-O-of-exec-d-processes.patch` | `b5f28865ef` | 2019-06-20 | **关键修复**：处理 exec 进程的阻塞 I/O |
| `0001-Send-exec-exit-event-on-failures.patch` | `c08d4da6e5` | 2019-06-28 | 失败时也发送 exec exit event |

**首次出现版本**: v19.03.x  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

**关键作用**: 
- `b5f28865ef` 防止 I/O 被子进程继承时导致 cleanup 永久阻塞
- 为 stream 增加 timeout 和 cancel 机制

---

### 4. Monitor 超时控制（2020-11）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-handleContainerExit-put-a-timeout-on-containerd-Dele.patch` | `05c20a6e1c` | 2020-11-02 | DeleteTask 增加超时，避免持锁挂起 |

**首次出现版本**: v20.10.x  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

---

### 5. Context Cancel 机制（2022-08）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-daemon-kill-exec-process-on-ctx-cancel.patch` | `4b84a33217` | 2022-08-22 | **核心修复**：context cancel 时 kill exec 进程 |
| `0001-daemon-Maintain-container-exec-inspect-invariant.patch` | `a09f8dbe6e` | 2022-08-24 | 维护 ExecCommands 的不变量，先删后清 Running |

**首次出现版本**: v23.0.x  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

**关键作用**:
- `4b84a33217` 使客户端 timeout/断开能触发服务端 exec 进程清理
- 解决拓展场景的核心问题之一

---

### 6. Client Disconnect 检测（2023-02）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-Fix-fd-leak-goroutine-when-attaching-stdin-only.patch` | `50d3028464` | 2023-02-21 | 修复 stdin-only attach 的 goroutine 泄漏 |
| `0001-Fix-goroutine-fd-leak-when-client-disconnects.patch` | `2d134c5abd` | 2023-02-21 | **核心修复**：检测 client disconnect 并强制关闭 stream |

**首次出现版本**: v24.0.x  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

**关键作用**:
- `2d134c5abd` 使用 epoll 监听 client fd，检测 EPOLLHUP/EPOLLERR
- 使 daemon 能感知客户端 timeout 断开
- 解决拓展场景的另一个核心问题

---

### 7. Race Condition 修复（2023-06）

| Patch 文件 | Commit | 时间 | 说明 |
|-----------|--------|------|------|
| `0001-daemon-fix-panic-on-failed-exec-start.patch` | `3b28a24e97` | 2023-06-22 | 修复 exec start 失败时的 panic |

**首次出现版本**: v25.0.x  
**来源仓库**: `/home/fsq/Desktop/fsq/moby-26.1.4/moby`

**关键作用**: 处理 `ProcessEvent` 和 `ContainerExecStart` 竞争时的 `execConfig.Process == nil` race

---

## 应用顺序建议

如果需要在 v1.13.1 上逐步应用这些修复，建议按以下顺序：

1. **基础修复** (2017)
   - `3cc0d6bb04` - 启动失败 cleanup
   - `2ddec97545` - Stream 重构
   - `84d6240cfe` - Attach race 修复

2. **I/O 处理** (2019)
   - `b5f28865ef` - 阻塞 I/O 处理
   - `c08d4da6e5` - 失败路径 exit event

3. **超时控制** (2020)
   - `05c20a6e1c` - DeleteTask 超时

4. **Context 机制** (2022)
   - `4b84a33217` - Context cancel kill
   - `a09f8dbe6e` - ExecCommands 不变量

5. **Client 检测** (2023)
   - `50d3028464` - Stdin goroutine 泄漏
   - `2d134c5abd` - Client disconnect 检测
   - `3b28a24e97` - Exec start race

**注意**: 这些 patches 依赖较新的 containerd API 和 Go 标准库特性，直接应用到 v1.13.1 可能需要调整。建议直接升级到 v26.1.4 或更高版本。

## 核心修复组合

解决拓展场景（高并发 + 客户端 timeout）的最小修复集合：

```
2d134c5abd (client disconnect 检测)
    +
4b84a33217 (ctx cancel kill exec)
    +
b5f28865ef (stream I/O timeout)
    +
3cc0d6bb04 (错误路径 cleanup)
```

这四个 commit 共同构成了从"客户端断开检测"到"daemon 侧进程清理"的完整链路。

## 文件统计

- 总 patch 数：12 个
- 涵盖时间：2017-01 至 2023-06（6.5 年）
- 总大小：约 136 KB
- 最大文件：`0001-Convert-script-shebangs-from-bin-bash-to-usr-bin-env.patch` (32 KB，merge commit)
- 核心修复：`2d134c5abd` (13 KB)、`0001-Move-attach-code-to-stream-package.patch` (18 KB)
