---
name: hermes-feishu-adapter
description: 配置 Hermes Gateway 飞书（Feishu/Lark）适配器，解决消息无法收发的问题。适用于 Windows + WSL 混合环境。
category: productivity
---

# Hermes 飞书适配器配置

## 触发条件
- 飞书适配器配置
- 飞书消息无法回复
- hermes gateway 启动失败
- 飞书白名单问题

## 步骤

### 1. 判断当前环境
先确认是在 Windows 还是 WSL Linux 环境里：
- Windows: hermes 命令直接可用，配置文件在 `%USERPROFILE%\.hermes\`
- WSL: hermes 命令找不到，实际程序跑在 Windows 上，WSL 只是终端

检查方法: `uname -a`（看是否有 Microsoft/WSL）

配置文件路径:
- Windows: `C:\Users\<用户名>\.hermes\`
- WSL: `/mnt/c/Users/<用户名>/.hermes/`

### 2. 检查 Gateway 运行状态
在 Windows 环境下执行（PowerShell 或 CMD）：
```bash
hermes gateway status
```
如果报错 "Not supported on this platform"，说明当前终端是 WSL，实际 Gateway 应该已经在 Windows 后台运行。

### 3. Windows 程序不能在 WSL 里直接调用
如果在 WSL 里执行 hermes 相关命令，报 `Command not found`：
这是正常的，因为 hermes 是 Windows 程序，WSL 环境下需要通过：
- 完整路径: `/mnt/c/Users/LENOVO/AppData/Roaming/npm/hermes`
- 或者先 `cd /mnt/c` 再调用

**重要**: Gateway 如果需要重启，应该在 Windows 侧操作，不能在 WSL 里 `hermes gateway start`

### 4. 飞书消息不回应 — 白名单问题
**现象**：飞书发消息给机器人没有回应

**排查**：日志里有大量 `No user allowlists configured` 警告

**原因**：.env 里的 `FEISHU_ALLOWED_USERS` 只有部门 ID，普通用户被拒绝

**解决方案**（按场景选）：
- 快速测试: `GATEWAY_ALLOW_ALL_USERS=true`（允许所有用户）
- 精确配置: 逐个加用户 ID 到 `FEISHU_ALLOWED_USERS`

配置路径: `%USERPROFILE%\.hermes\.env`

### 5. 重启 Gateway 遇到 PID 锁冲突
**现象**：杀进程后重启失败，日志显示 OSError（PID 无效）

**原因**：`gateway.pid` 文件里记录的 PID 已不存在，但锁文件没清理

**解决**：删除 `%USERPROFILE%\.hermes\gateway.pid` 再重启

Windows 命令: `del %USERPROFILE%\.hermes\gateway.pid`

### 6. 配对飞书账号
获取配对码后执行（Windows 下）：
```bash
hermes pairing approve feishu <PAIRING_CODE>
```
配对成功后飞书账号会收到确认，ou_xxx 开头的用户 ID 被授权。

### 7. 验证配置生效
重启后确认：
- `hermes gateway status` 显示 running
- 端口 18789 正常监听
- 飞书发消息有回应

## 避坑指南
- WSL 环境下找不到 hermes 命令是正常的，不要尝试在 WSL 里安装
- 修改 .env 后需要重启 Gateway 才能生效
- PID 锁冲突时只删 .pid 文件，不要删 .env
- `GATEWAY_ALLOW_ALL_USERS=true` 是测试用，生产环境建议精确配置白名单
