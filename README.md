# mu-self-evolve

> AI Agent 持续进化系统 — 让 Agent 像人一样从经验中学习

## 这是什么

mu-self-evolve 是一个为 AI Agent（基于 LLM 的智能助手）设计的持续进化系统。它通过**每日经验沉淀 + 每周错误反思**的双周期节律，将散落的对话经验、踩坑记录、木老师纠正转化为结构化的持久记忆，让 Agent 在不更新模型参数的前提下持续变强。

核心能力：
- **经验三分法**：LEARNINGS（经验）、ERRORS（错误）、FEATURE_REQUESTS（功能需求）分类记录
- **VFM 确定性验算**：经验提升前必须通过脚本验证（复现次数、跨任务数、解决率），而非仅靠 LLM 自评
- **评分+衰减淘汰**：用 `effective_score = VFM × 0.5^(age/30)` 替代行数硬截断，保留高价值记忆
- **WHERE×WHY 病理归档**：按"出错环节×根因类型"结构化归档，而非模糊的叙事
- **主动 Skill 合成**：当同类经验积累到阈值时，自动生成新 Skill 草案
- **评委偏差审计**：每周淘汰前做缺陷注入自检，防止淘汰机制静默失效

## 快速开始

### 安装

```bash
# 克隆仓库
git clone https://github.com/muippt/mu-self-evolve.git
cp -r mu-self-evolve <SKILL_DIR>/mu-self-evolve
```

### 初始化

首次运行前，执行初始化脚本创建目录结构：

```bash
cd <SKILL_DIR>/mu-self-evolve
# 执行初始化（自动检测环境：OpenClaw 或 Claude Code）
bash -c '
if [ -d "$HOME/.openclaw/workspace" ]; then
  WS="$HOME/.openclaw/workspace"
  LEARNINGS_DIR="$WS/.learnings"
  NARRATIVES_DIR="$WS/memory/narratives"
  ARCHIVE_DIR="$WS/memory/archive"
elif [ -d "$HOME/.claude" ]; then
  WS="$HOME/.claude/memory"
  LEARNINGS_DIR="$WS/.learnings"
  NARRATIVES_DIR="$WS/narratives"
  ARCHIVE_DIR="$WS/archive"
else
  WS="$HOME/.openclaw/workspace"
  LEARNINGS_DIR="$WS/.learnings"
  NARRATIVES_DIR="$WS/memory/narratives"
  ARCHIVE_DIR="$WS/memory/archive"
fi
mkdir -p "$LEARNINGS_DIR/archive" "$NARRATIVES_DIR" "$ARCHIVE_DIR"
for f in LEARNINGS.md ERRORS.md FEATURE_REQUESTS.md; do
  [ ! -f "$LEARNINGS_DIR/$f" ] && echo "# ${f%.md} Log" > "$LEARNINGS_DIR/$f"
done
echo "✅ 初始化完成 | WS=$WS"
'
```

### 设置定时任务

```bash
# OpenClaw
openclaw cron add \
  --name "🧬 每日自我进化" \
  --cron "30 19 * * *" \
  --tz "Asia/Shanghai" \
  --message "执行每日自我进化流程（见 mu-self-evolve SKILL.md）。今天是周几？如果是周五，额外执行【第六步：周度反思】。"

# Claude Code（系统 crontab）
crontab -e
# 添加：
30 19 * * * cd ~/.claude && claude --prompt "执行每日自我进化流程（见 mu-self-evolve SKILL.md）。"
```

## 工作流程

### 每日六步

```
读取今日日记
    ↓
① 读取日记 → 提取事件
    ↓
② 分类录入 → LEARNINGS/ERRORS/FEATURE_REQUESTS（含冲突裁决+病理标注）
    ↓
③ 检查提升条件 → VFM 验算脚本（确定性验证）
    ↓
④ 更新长期记忆 → MEMORY.md（≤80行，超限触发 mini 蒸馏）
    ↓
⑤ 子Agent质量回顾
    ↓
⑥ 发送摘要
```

### 周五加跑：周度反思

```
6a. 错误扫描 → 提取重复模式
6b. Narrative → (WHERE×WHY) 病理矩阵
6c. 规则提议 → 重复≥2次的错误提炼为行为规则
6d. MEMORY.md 蒸馏 → 精简到≤80行
6e. 归档旧日志 → 30天+的日记/叙事移入 archive
6f. 评分+衰减淘汰 → bias_audit + eviction_score（替代行数硬截断）
6g. 条目聚类+Skill合成 → ≥3条同类→生成Skill草案
```

## 文件结构

```
<AGENT_HOME>/workspace/           # 或 ~/.claude/memory/（Claude Code）
├── MEMORY.md                    # 长期记忆（≤80行）
├── SOUL.md                      # 行为风格、原则（可选）
├── TOOLS.md                     # 工具配置（可选）
├── AGENTS.md                    # 工作流规则（可选）
├── memory/
│   ├── YYYY-MM-DD.md            # 每日日记
│   ├── narratives/              # 周度叙事
│   └── archive/                 # 30天+归档
└── .learnings/
    ├── LEARNINGS.md             # 经验记录（≤500行）
    ├── ERRORS.md                # 错误记录（≤300行）
    ├── FEATURE_REQUESTS.md      # 功能需求
    └── archive/                 # 已提升/已解决/dormant归档
```

## VFM 评分体系

经验提升为永久记忆前，需通过四维加权评分（≥50分才提升）：

| 维度 | 权重 | 说明 |
|------|------|------|
| 高频使用 | ×3 | 这条规则会每天/每周触发吗？ |
| 减少失败 | ×3 | 能把之前的翻车变成成功吗？（需脚本验算数据） |
| 减轻木老师负担 | ×2 | 能让木老师少解释一句话吗？ |
| 节省未来成本 | ×2 | 能让未来的我省时间/token吗？ |

## 淘汰机制

v3.0 用评分+衰减淘汰替代行数硬截断：

```
effective_score = VFM × 0.5^(days_since_last_seen / 30)
```

- 半衰期 30 天（对标 CrewAI）
- effective_score < 25 且 age > 14天 → 建议归档
- Recurrence≥2 的条目豁免（跨任务复现覆盖低分）
- 被检索命中时 Last-Seen 重置 → 衰减重置

## 环境兼容

| 环境 | 路径 | 调度 |
|------|------|------|
| **OpenClaw**（主） | `<AGENT_HOME>/workspace/` | `openclaw cron` |
| **Claude Code** | `~/.claude/` | 系统 crontab |
| **其他** | 自动检测 | 手动触发 |

所有 Python 脚本通过 `env_detect.py` 自动检测环境，无需手动指定路径。

## 脚本说明

| 脚本 | 用途 |
|------|------|
| `scripts/env_detect.py` | 环境自动检测（OpenClaw / Claude Code） |
| `scripts/vfm_verify.py` | VFM 确定性验算（复现/跨任务/解决率/病理集中度→置信度） |
| `scripts/eviction_score.py` | 评分+衰减淘汰计算（effective_score） |
| `scripts/bias_audit.py` | 评委偏差审计（缺陷注入→验证淘汰机制可信度） |

```bash
# 验算某条经验规则
python3 scripts/vfm_verify.py mcp.token_expired

# 运行淘汰评分（无参数自动检测环境）
python3 scripts/eviction_score.py

# 评委偏差审计
python3 scripts/bias_audit.py
```

## 学术参考

本系统的设计借鉴了以下研究：

- **ExpeL** (arXiv:2308.10144) — 经验提取双层检索（案例+规则）
- **Reflexion** (arXiv:2303.11366) — 语言化反思作为"语义梯度"
- **Voyager** (arXiv:2305.16291) — 可执行代码技能库 + 自动课程
- **MemGPT** (arXiv:2310.08560) — OS 式虚拟内存管理
- **Generative Agents** (arXiv:2304.03442) — 观察→反思→规划三层记忆
- **GSME** (2025) — 提议与授信分离（LLM诊断+确定性验证）
- **The Blind Curator** (2025) — 评委偏差与淘汰机制静默失效

## License

MIT
