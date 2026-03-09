---
name: deep-research
description: "使用 Gemini Deep Research Agent 进行自主深度研究，生成带引用的详细研究报告。当用户需要深度调研话题、生成研究报告、或需要多轮搜索和分析时使用。触发词：深度研究、deep research、帮我调研、研究一下、写研究报告、deep dive、详细分析。"
---

# Deep Research Agent

使用 Gemini Deep Research Pro 执行自主多步搜索-阅读-分析的深度研究，通常需要 5-20 分钟完成。

## 工作流程

### 1. 确认研究主题

- 与用户确认研究主题和范围
- 如果话题模糊，帮助用户明确研究方向和关注点
- 将用户需求转化为清晰的英文或中文研究查询

### 2. 获取 API Key

从 `openclaw.json` 配置中读取 Gemini API Key：

```bash
# 路径：skills.entries.nano-banana-pro.apiKey
# 或：agents.defaults.memorySearch.remote.apiKey
```

使用 `exec` 工具从配置文件提取：
```bash
node -e "const c=JSON.parse(require('fs').readFileSync('/Users/feifei/.openclaw/openclaw.json','utf8')); console.log(c.skills?.entries?.['nano-banana-pro']?.apiKey || c.agents?.defaults?.memorySearch?.remote?.apiKey || '')"
```

### 3. 执行研究

**提醒用户**：深度研究通常需要 5-20 分钟，请耐心等待。

运行脚本：
```bash
GEMINI_API_KEY="<api_key>" node /Users/feifei/.openclaw/workspace/skills/deep-research/deep-research.mjs "<研究主题>"
```

可选参数：
- `--timeout <seconds>` — 超时时间，默认 3600 秒（1 小时）
- `--no-stream` — 使用轮询模式（如果流式出错可尝试）

**重要**：由于研究耗时较长，使用 exec 工具时设置足够的超时时间（至少 1200 秒）。

### 4. 后续追问

研究完成后，脚本会输出 Interaction ID。如果用户需要追问或深入某个方面：

```bash
GEMINI_API_KEY="<api_key>" node /Users/feifei/.openclaw/workspace/skills/deep-research/deep-research.mjs --follow-up <interaction_id> "<追加问题>"
```

### 5. 呈现结果

- stdout 输出为 Markdown 格式的研究报告
- 保持原始 Markdown 格式呈现给用户
- 报告包含引用来源链接
- 如果用户需要，可以将报告保存为文件

## 输出说明

- **stdout** — 最终研究报告（Markdown 格式）
- **stderr** — 进度信息：
  - 🔬 研究开始
  - 📊 状态更新
  - 💭 思考摘要（Agent 的中间思考过程）
  - ✅ 研究完成 + 耗时
  - 📎 Interaction ID（用于后续追问）

## 错误处理

| 退出码 | 含义 | 处理方式 |
|--------|------|----------|
| 0 | 成功 | 正常呈现报告 |
| 1 | 参数错误 | 检查研究主题是否提供 |
| 2 | API 错误 | 检查 API Key 是否有效，提示用户检查配额 |
| 3 | 超时 | 建议缩小研究范围或增加超时 |

## 安全

- 不要将 API Key 直接展示给用户
- 使用环境变量传递 API Key，不要硬编码在命令中显示
