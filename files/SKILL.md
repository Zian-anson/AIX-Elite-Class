---
name: hermes-modelscope-delegation
description: Hermes 子Agent通过魔搭（ModelScope）API路由配置 — 多模型分工、 delegation 配置、魔搭模型实测清单。适用场景：需要派出子Agent并行执行任务、用魔搭免费额度（2000次/天）降低API成本、针对不同任务类型选择不同模型。
author: Hermes Agent
tags: [hermes, modelscope, delegation, multi-agent, api-routing]
homepage: https://www.modelscope.cn
related_skills: [hermes-agent]
---

# Hermes 魔搭（ModelScope）子Agent路由方案

## 背景

主Agent（MiniMax-M2.7，通过 minimax-cn 官方付费API）可派出子Agent，通过魔搭免费API（2000次/天）执行任务，实现多模型分工协作。

**关键约束：MiniMax-M2.7在魔搭上返回空响应，必须用M2.5或其他模型。**

## 核心配置

### 1. delegation 段（~/.hermes/config.yaml）

```yaml
delegation:
  model: MiniMax/MiniMax-M2.5
  provider: custom
  base_url: https://api-inference.modelscope.cn/v1
  api_key: ms-你的魔搭token
  max_iterations: 50
  reasoning_effort: low
  default_toolsets:
    - terminal
    - file
    - web
```

### 2. custom_providers 段（~/.hermes/config.yaml）

```yaml
custom_providers:
  - name: Api-inference.modelscope.cn
    base_url: https://api-inference.modelscope.cn/v1
    api_key: ms-你的魔搭token
    model: MiniMax/MiniMax-M2.5
```

## 魔搭模型实测清单（2026-04-14）

### ✅ 可用模型

| 模型ID | 用途 | 备注 |
|--------|------|------|
| `deepseek-ai/DeepSeek-V3.2` | 综合默认 | **推荐默认**，实测稳定 |
| `moonshotai/Kimi-K2.5` | 综合 | 实测稳定 |
| `deepseek-ai/DeepSeek-R1-Distill-Qwen-7B` | 深度推理/分析 | 推理能力强 |
| `Qwen/Qwen3-Coder-30B-A3B-Instruct` | **代码任务** | Qwen代码家族 |
| `MiniMax/MiniMax-M2.5` | MiniMax系默认 | M2.7在魔搭上返回空 |

### ❌ 不工作模型（返回空choices）

- `MiniMax/MiniMax-M2.7` — 魔搭上不工作
- `Qwen/Qwen3-8B` — 返回空
- `ZhipuAI/GLM-4.7-Flash` — 返回空

## 使用方式：delegate_task

### 方式A：直接调用（用户偏好）

```
delegate_task(
  goal="任务描述",
  model="deepseek-ai/DeepSeek-V3.2",  # 可选，默认用delegation配置的
  provider="custom"                      # 必填，指向魔搭
)
```

### 不同任务的模型选择

| 任务类型 | 推荐模型 | 示例 |
|----------|----------|------|
| 代码编写/重构 | `Qwen/Qwen3-Coder-30B-A3B-Instruct` | 快速代码生成 |
| 代码审查/安全 | `deepseek-ai/DeepSeek-R1-Distill-Qwen-7B` | 深度分析 |
| 调研/信息汇总 | `deepseek-ai/DeepSeek-V3.2` | 综合能力强 |
| 中文写作/摘要 | `moonshotai/Kimi-K2.5` | 中文优化 |
| 默认/轻量任务 | `MiniMax/MiniMax-M2.5` | delegation默认 |

## API信息

- **OpenAI兼容端点**: `https://api-inference.modelscope.cn/v1`
- **Anthropic兼容端点**: `https://api-inference.modelscope.cn`
- **Token格式**: `ms-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- **免费额度**: 每天2000次
- **注册地址**: https://www.modelscope.cn/my/myaccesstoken

## 踩坑记录（实测经验）

### ⚠️ MiniMax-M2.7 在魔搭上返回空
- **现象**：`choices: null`，API 认证通过但模型无输出
- **原因**：魔搭上的 M2.7 模型版本与官方 API 不兼容
- **解决**：改用 `MiniMax/MiniMax-M2.5` 或其他模型
- **影响范围**：魔搭 custom_provider 默认模型也需同步改为 M2.5

### ⚠️ config.yaml 编码污染（反复踩坑）
- **现象**：在 WSL 里用 `cat > file`、`echo`、`heredoc` 重定向等方式编辑 config.yaml，写入后 Hermes 报 `unacceptable character #x0000`
- **原因**：WSL 环境向 Windows 文件系统写入时混入 UTF-16LE 零宽字符
- **解决**：
  1. 恢复备份：`cp config.yaml.bak config.yaml`
  2. 用 Python 直接读写：`python3 -c "open(path, 'w').write(content)"`
  3. 用 Windows 工具编辑
- **验证干净**：Linux 下 `file config.yaml` 应显示 `ASCII text` 或 `UTF-8 text`，不是 `data`

## 注意事项

1. **文件编码**：绝对不要用 WSL 的 cat/heredoc/echo 重定向写入 config.yaml，每次都会污染
2. **模型验证**：魔搭模型会更新，不保证列表永久可用，以实际 curl 测试为准
3. **主Agent**：主Agent用 MiniMax 官方付费 API（minimax-cn provider），子Agent用魔搭免费 API，分工明确
4. **备份习惯**：每次编辑 config.yaml 前先备份 cp config.yaml config.yaml.bak
