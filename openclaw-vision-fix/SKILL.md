---
name: openclaw-vision-fix
description: "诊断和修复 OpenClaw 的图片识别（Vision）问题。OpenClaw 用独立的 imageModel 处理图片，主模型无需支持 vision。"
version: 1.0.0
---

# OpenClaw Vision 修复

## 架构理解
OpenClaw 有 **两个独立的模型配置**：
- `agents.defaults.model.primary` — 主聊天模型（如 DeepSeek V4 Flash）
- `agents.defaults.imageModel.primary` — 图片理解模型（单独的）

即使主模型不支持 vision，OpenClaw 也会自动将图片 offload 给 imageModel 处理。

## 诊断步骤

### 1. 检查当前配置
```bash
cat ~/.openclaw/openclaw.json | python3 -c "
import json,sys
d=json.load(sys.stdin)
print('主模型:', d['agents']['defaults']['model'].get('primary',''))
print('图片模型:', d['agents']['defaults']['imageModel'].get('primary',''))
print('Provider:', list(d['models']['providers'].keys()))
"
```

### 2. 检查模型是否声明了图片支持
看 `openclaw.json` 里对应模型的 `input` 字段是否包含 `"image"`。

### 3. 检查可用认证
```bash
openclaw models auth list
```

### 4. 用 CLI 测试视觉模型
```bash
openclaw infer image describe --model <模型ID> --file <图片路径> --prompt '描述这张图' --timeout-ms 60000
```

### 5. 检查日志
```bash
tail -60 ~/Library/Logs/openclaw/gateway.log
```

## 修复方案

### 方案 A：切换图片模型（推荐）
```bash
# 先找到可用的视觉模型
openclaw models search visual
# 设置图片模型
openclaw models set-image <模型ID>
```

### 方案 B：直接改配置文件
```bash
# 先备份
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak
# 修改 agents.defaults.imageModel.primary 的值
```

## 注意事项
- DeepSeek V4 API 目前不支持原生 image input（ChatCompletions API 只有 text messages）
- 不要强行改主模型的 input 为 ["text","image"] — 会导致 API 报错
- 切换后需要重启 OpenClaw Gateway 才能生效
- 图片文件存储在 `~/.openclaw/media/inbound/`
