# Hermes-Agent 安装实操步骤（可直接复刻）

## 一、核心前提

全程必须使用**管理员权限**终端操作，权限不足会直接导致安装失败。

***

## 二、具体安装步骤

### 步骤1：打开管理员权限终端

1. 按下 `Win+X` 组合键
2. 按系统版本选择：
   - Windows 10：选择**Windows PowerShell (管理员)**
   - Windows 11：选择**终端 (管理员)**，再切换到 PowerShell 标签页

***

### 步骤2：为 PowerShell 配置 Clash 代理（解决网络问题）

在管理员 PowerShell 中**逐行输入**以下命令，每行输完按回车执行：

```powershell
$env:http_proxy="http://127.0.0.1:你的代理"
$env:https_proxy="http://127.0.0.1:你的代理"
```

#### 网络连通性验证

输入以下命令并回车，检查网络是否通畅：

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1" -UseBasicParsing
```

✅ 正常结果：返回脚本完整内容、无报错 → 可进入下一步

***

### 步骤3：放开 PowerShell 脚本执行权限（必做）

Windows 默认禁止运行外来 ps1 脚本，执行以下命令解除限制：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

弹出确认提示时，输入**Y**，按回车确认。

***

### 步骤4：执行安装指令

```
curl -fsSL <https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh> | bash

```

***

### 步骤5：验证安装是否成功

1. 关闭当前 PowerShell 窗口
2. **重新打开管理员权限 PowerShell**
3. 输入以下命令并回车：

```powershell
hermes --version
```

✅ 安装成功：命令正常输出版本号

***

## 三、通用安装提示

安装全程遇到权限、确认类弹窗，统一输入**Y**并回车即可继续。

## 四、Word文档使用方法

1. 复制以上全部内容
2. 打开 Microsoft Word，新建空白文档
3. 粘贴内容，按需调整字体、行距
4. 保存为 `.docx` 格式文档

> （注：文档部分内容可能由 AI 生成）

