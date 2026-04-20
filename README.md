# Claude Summarize Skill

自动将 Claude Code 对话归档为精美的 HTML 能力分析报告。

<img width="682" height="776" alt="image" src="https://github.com/user-attachments/assets/f4a11301-30b6-4ac3-a505-8c09a71daa1e" />

<img width="690" height="555" alt="image" src="https://github.com/user-attachments/assets/554d5e77-ca6a-4af2-8a70-86c152879825" />


## 功能

- **Session 去重** — 已归档的对话自动跳过，避免重复处理
- **7 维能力雷达** — 可视化展示需求拆解、技术选型、代码实现、调试排错、审美设计、系统思维、AI 协同
- **结构化总结** — 每段对话提取任务描述、最终结果、关键问题、能力体现、待提升点
- **分类筛选** — Dev / Test / Q&A / Config / Design / Learn / Ops 七类标签
- **AI 话术宝典** — 自动检测模糊表达，给出更专业的 prompt 建议
- **Worth Trying** — 随机推荐进阶挑战，保持学习动力

## 安装

```bash
# 1. 克隆仓库
git clone https://github.com/yourname/claude-summarize-skill.git

# 2. 将 skill 文件放入 Claude Code 命令目录
cp claude-summarize-skill/summarize.md ~/.claude/commands/

# 3. 确保已安装 jq（解析 JSONL 用）
# macOS
brew install jq
# Ubuntu/Debian
sudo apt-get install jq
```

重启 Claude Code 后，输入 `/summarize` 即可调用。

## 配置（可选）

| 环境变量 | 说明 | 默认值 |
|---------|------|--------|
| `CLAUDE_ARCHIVE_OUTPUT` | HTML 归档文件输出路径 | `./claude-archive/index.html` |
| `CLAUDE_PROJECT_NAME` | Claude Code project 名称 | 自动计算（当前目录编码） |

### 示例：集中归档到统一目录

```bash
export CLAUDE_ARCHIVE_OUTPUT="$HOME/Documents/claude-archive/index.html"
```

### 示例：手动指定 project 名称

Claude Code 的 project 目录位于 `~/.claude/projects/` 下，名称是工作目录路径的编码：

| 工作目录 | Project 名称 |
|---------|-------------|
| `/Users/alice/my-project` | `-Users-alice-my-project` |
| `/home/bob/code` | `-home-bob-code` |
| `C:\Users\Carol\work` | `C-Users-Carol-work` |

```bash
export CLAUDE_PROJECT_NAME="-Users-alice-my-project"
```

## 使用

在任意已初始化为 Claude Code project 的目录中：

```
/summarize
```

Skill 会自动：
1. 扫描 `~/.claude/projects/<project-name>/*.jsonl` 中的对话记录
2. 读取现有的 HTML 归档（如有），提取已处理的 session ID
3. 仅对新 session 生成结构化总结
4. 更新雷达图分数、统计数字、Worth Trying 内容
5. 输出/更新 HTML 文件

打开生成的 HTML 即可在浏览器中查看完整的可视化报告。

## 依赖

- [Claude Code](https://claude.ai/code) CLI
- [jq](https://jqlang.github.io/jq/)（命令行 JSON 处理器）

## 文件结构

```
.
├── README.md
├── summarize.md          # Skill 定义文件（放入 ~/.claude/commands/）
├── example-archive.html  # 示例输出（供预览参考）
└── docs/
    └── configuration.md  # 高级配置说明
```

## 自定义样式

生成的 HTML 采用 Apple 简约科技风：
- 黑白灰 + Apple 蓝（#0071e3）配色
- 无圆角卡片、无左侧/顶部边框
- 响应式布局，支持移动端

如需修改样式，直接编辑生成后的 HTML `<style>` 标签即可。

## License

MIT
