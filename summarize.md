---
name: summarize
description: Summarize Claude Code conversations into a beautiful HTML archive with radar chart abilities, category filtering, session deduplication, and AI collaboration tips.
allowed-tools: Bash Read Edit Write
---

## 配置（执行前自动检测）
1. 通过 Bash 执行 `echo "${CLAUDE_ARCHIVE_OUTPUT:-./claude-archive/index.html}"` 确定 HTML 输出路径（`$OUTPUT_PATH`）
2. 通过 Bash 检测 project 名称：
   - 若 `CLAUDE_PROJECT_NAME` 已设置，直接使用
   - 否则执行 `pwd | tr '/' '-'` 自动计算 project 名称（`$PROJECT_NAME`）
   - 验证 `~/.claude/projects/$PROJECT_NAME/` 是否存在
   - 若不存在，执行 `ls ~/.claude/projects/` 列出所有目录，询问用户选择正确的 project 名称
3. 执行 `mkdir -p $(dirname "$OUTPUT_PATH")` 确保输出目录存在

## 目标
将 `~/.claude/projects/$PROJECT_NAME/*.jsonl` 中尚未归档的 session 生成结构化总结，更新到 `$OUTPUT_PATH`。

## 关键机制：Session 去重
1. 读取 HTML 中 `<script type="application/json" id="processed-sessions">` 的 JSON 数组
2. 扫描 `~/.claude/projects/$PROJECT_NAME/*.jsonl`，按修改时间排序
3. **仅处理不在已处理列表中的 session**，跳过已归档的以节省 token
4. 处理完成后，将新 session ID 追加到 `processed-sessions` JSON 中

## 对话分类标签
每个 session 归入以下分类之一，在卡片 `data-category` 属性和标签上显示：
- **dev** (Dev / 开发)：编写代码、构建项目、功能开发
- **test** (Test / 测试)：测试代码、修复 bug、性能调优
- **qa** (Q&A / 问答)：技术咨询、概念解释、方案讨论
- **config** (Config / 配置)：安装软件、配置环境、设置 IDE
- **design** (Design / 设计)：UI/UX 设计、视觉创作
- **learn** (Learn / 学习)：学习新技术、阅读文档
- **ops** (Ops / 运维)：部署、CI/CD、工具封装、自动化

## 能力模型（7 维雷达图 + 两栏布局）
每次更新时，根据本次及历史对话重新评估 7 维得分（0-100）：
1. 需求拆解 / Decomposition
2. 技术选型 / Tooling
3. 代码实现 / Implementation
4. 调试排错 / Debugging
5. 审美设计 / Aesthetics
6. 系统思维 / Architecture
7. AI 协同 / AI Co-pilot

### 布局规范（两栏，圆角卡片包裹）
- 使用 `.ability-card` 圆角矩形包裹，`border-radius: 24px`
- 两栏网格：`grid-template-columns: 1fr 1fr; gap: 48px;`
- **左栏**：Canvas 雷达图（360x360，半径 120，7 维双语标签）
- **右栏**：`.ability-summary` 包含两个 `.summary-col`
  - 上半部分：当前优势（3+ 项）
  - 下半部分：待加强（3+ 项）
- 每项包含：绿/红标签（维度名 + 分数）、小标题（一句话）、详细说明（1-2 句）
- 语言真诚、有洞察力，避免模板化套话

## Worth Trying / 值得尝试
紧跟在两栏布局下方。预写约 20 个 challenge 对象，每次随机显示 4 个不重复项。按钮 `Shuffle / 换一批`。

- 网格：`grid-template-columns: repeat(2, 1fr)`（桌面），`1fr`（手机）
- 每项：标题 + 描述 + 能力标签

## AI 话术宝典（可选板块）
阅读对话时，分析用户与 AI 协作的表达方式。发现不够专业/高效的表述时，在对应 session card 中新增 `.tips-box`：

```html
<div class="tips-box">
    <h4>AI 话术宝典 / Collaboration Tips</h4>
    <ul>
        <li><span class="before">"帮我做个好看点的"</span> <span class="after">→ "采用圆角 16px、纯白卡片、柔和阴影，参考 Apple 官网风格"</span></li>
    </ul>
</div>
```

**判定标准**：
- 模糊形容词（"好看"、"专业"、"高级"）无具体参数
- 反馈无具体位置或期望结果
- 缺少约束条件（技术栈、尺寸、配色等）
- 未发现则**不添加此板块**

## HTML 模板规范（Apple 科技风）
- **配色**：黑白灰 + Apple 蓝（#0071e3）。禁止紫色。
- **禁止**：`border-left` 或 `border-top` 样式（板块分隔线除外）
- **容器宽度**：`max-width: 980px`
- **大标题**：`~/vibe-coding/archive`，无副标题
- **板块标题**：`英文 / 中文` 格式
  - `Ability Radar / 能力雷达`
  - `Session Analytics / 会话分析`
  - `Conversation Archive / 会话归档`
- **卡片内文本**：`英文 / 中文` 格式
  - `Task / 任务描述`、`Outcome / 最终结果`、`Strengths / 能力体现`
- **统计标签**：`Sessions / 对话`、`Projects / 项目`、`Issues / 问题`、`Domains / 领域`
- **筛选按钮**：`All / 全部`、`Dev / 开发`、`Test / 测试`、`Q&A / 问答`、`Config / 配置`、`Design / 设计`、`Learn / 学习`、`Ops / 运维`
- **filter bar 间距**：`margin-bottom: 24px`

## 执行流程
1. 读取环境变量确定 `$OUTPUT_PATH` 和 `$PROJECT_NAME`
2. 读取 HTML 中的 `processed-sessions` JSON
3. 找出未处理的 session JSONL 文件
4. 对每个未处理 session，用 `jq` 提取用户消息理解任务
5. 分析对话中的 AI 协作表达方式，生成 AI 话术宝典（如发现问题）
6. 生成分类、任务描述、结果、问题解决、能力分析
7. 重新计算 7 维雷达图分数
8. 生成结构化能力总结（优势 3+ 项 + 待加强 3+ 项）
9. 更新 Worth Trying 内容
10. 插入新 session-card 到 `#session-list` 顶部
11. 更新统计数字、雷达图数据、processed-sessions、时间戳
