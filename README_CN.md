<p align="center">
  <img alt="mu-self-evolve" src="assets/default-banner.png" width="100%">
</p>

# mu-self-evolve

> AI Agent 持续进化系统 — 每日经验沉淀 + 每周错误反思，让 Agent 像人一样从经验中学习。

[English](README.md) | **中文** | [🌐 在线主页](https://muippt.github.io/mu-self-evolve/)

[![微信公众号](https://img.shields.io/badge/muippt-07C160?logo=wechat&logoColor=white)](https://mp.weixin.qq.com/s/v1JSZvlN5fvbOOHvkvXEtA)
[![小红书](https://img.shields.io/badge/muippt-FF2442?logo=xiaohongshu&logoColor=white)](https://xhslink.com/m/ESxtgUNMdl)
[![书籍](https://img.shields.io/badge/书籍-图解团队管理-BBDDE5?logo=bookstack&logoColor=white)](https://item.m.jd.com/product/14547345.html)
[![License](https://img.shields.io/github/license/muippt/mu-self-evolve)](LICENSE)
[![Version](https://img.shields.io/github/v/release/muippt/mu-self-evolve)](https://github.com/muippt/mu-self-evolve/releases)
[![Stars](https://img.shields.io/github/stars/muippt/mu-self-evolve)](https://github.com/muippt/mu-self-evolve/stargazers)

### 💡 使用场景示例

🧠 **每日经验沉淀** — 自动从当天对话中提取纠正、错误、正向反馈，分类录入结构化文件。

🔍 **VFM 确定性验算** — 经验提升为永久记忆前，必须通过脚本验证复现次数、跨任务数、解决率，而非仅靠 LLM 自评。

🪶 **评分+衰减淘汰** — 记忆条目按 30 天半衰期公式自动衰减，低价值条目自动归档，高复现条目豁免。

🏷️ **WHERE×WHY 病理归档** — 每条错误标注「出错环节×根因类型」，结构化分析模式而非模糊叙事。

🧪 **评委偏差审计** — 注入已知属性的合成测试条目，验证淘汰机制是否正常工作，防止「盲人策展人」问题。

🛠️ **主动 Skill 合成** — 当同一领域≥3条经验聚类时，自动生成 Skill 草案（骨架+脚本框架），交人工确认。

⏰ **定时自动化** — 设置每日 cron 任务自动运行六步进化流程，周五额外执行周度反思。

📊 **跨环境支持** — 自动检测 OpenClaw、Claude Code 或其他 LLM Agent 环境，适配路径和调度方式。

---

### ✨ 核心亮点

#### VFM 确定性验算

传统方案依赖 LLM 自评来决定哪些经验值得提升，这不可靠——LLM 可能产生幻觉式自信。mu-self-evolve 实现了「提议与验证分离」（受 GSME, 2025 启发）：

| 步骤 | 执行者 | 做什么 |
|------|--------|--------|
| 提议 | LLM | 识别模式，提议规则 |
| 验证 | 脚本（`vfm_verify.py`） | 统计复现次数、跨任务数、解决率、病理集中度 → 输出置信度评分 0–100 |
| 提升 | 门控 | 置信度≥70 + 复现≥2 + VFM≥50 → 提升到永久记忆 |

#### 评分+衰减淘汰

用时间衰减评分模型替代行数硬截断（受 CrewAI 半衰期衰减启发）：

```
effective_score = VFM × 0.5^(days_since_last_seen / 30)
```

| 状态 | 条件 | 处理 |
|------|------|------|
| 已提升 | status=promoted | 移至永久记忆 |
| 已解决+旧 | status=resolved 且 >30天 | 归档 |
| 高复现 | Recurrence≥2 | **豁免**（不论分数和年龄） |
| 仅1次+旧 | Recurrence=1 且 >14天 | 标记 dormant，归档 |
| 低分+旧 | effective<25 且 >14天 | 归档 |
| 正常 | effective≥25 或 <14天 | 保留 |

#### WHERE×WHY 病理键

每条错误和经验记录都标注结构化病理键：

**WHERE 标签**（出错环节）：`skill_load` · `mcp_call` · `file_write` · `api_call` · `browser_op` · `memory_op` · `agent_dispatch` · `prompt_parse` · `auth` · `other`

**WHY 标签**（根因类型）：`param_missing` · `token_expired` · `timing_race` · `perm_denied` · `knowledge_gap` · `format_mismatch` · `timeout` · `logic_error` · `config_drift` · `user_correction` · `other`

两组标签均支持用户自定义——随着 Agent 遇到新环境，你可以添加自己的领域标签。

#### 评委偏差审计（盲人策展人防御）

受 "The Blind Curator"（2025）启发——如果淘汰机制本身存在偏差，它会在操作者以为系统正常工作的同时，静默地清除有价值条目。`bias_audit.py` 脚本：

1. 创建 5 条已知属性的合成测试条目（应保留、应归档、应豁免等）
2. 用实际淘汰逻辑运行
3. 对比实际结果与预期结果
4. 报告 PASS/FAIL——任何 FAIL 意味着淘汰逻辑有 bug

---

### 📌 与同类工具对比

| 能力 | mu-self-evolve | 裸记忆文件 | Mem0 | Zep |
|------|---------------|-----------|------|-----|
| 每日自动沉淀 | ✅ 六步工作流 | ❌ 手动 | ❌ 仅 API | ❌ 仅 API |
| 确定性验算 | ✅ 脚本验证 | ❌ | ✅ LLM-as-editor | ❌ |
| 时间衰减淘汰 | ✅ 30天半衰期 | ❌ | ✅ 半衰期 | ✌️ 双时间线 |
| 病理归档（WHERE×WHY） | ✅ 结构化标签 | ❌ | ❌ | ❌ |
| 评委偏差审计 | ✅ 缺陷注入 | ❌ | ❌ | ❌ |
| 主动 Skill 合成 | ✅ 聚类→草案 | ❌ | ❌ | ✌️ Dreaming |
| 跨环境支持 | ✅ OpenClaw + Claude Code | 不适用 | ❌ | ❌ |
| 无需外部 API | ✅ 完全本地 | ✅ | ❌ | ❌ |

---

### 🚀 五大工作流

| 工作流 | 场景 | 触发方式 |
|--------|------|---------|
| 每日进化（步骤1-6） | 提取今日经验、验算、提升、发摘要 | 定时 cron（每日） |
| 周度反思（步骤6a-6g） | 错误扫描、叙事生成、淘汰、Skill合成 | 周五（与每日合并） |
| 即时记录 | 用户纠正 Agent 或错误发生 | 触发检测（实时） |
| 手动验算 | 检查某条规则的置信度 | `python3 vfm_verify.py <pattern-key>` |
| 淘汰审计 | 运行偏差测试+评分清理 | `python3 bias_audit.py && python3 eviction_score.py` |

---

### ⚙️ 技术规格

| 项目 | 说明 |
|------|------|
| 运行时 | Python 3.8+（仅脚本）；任意 LLM Agent（OpenClaw / Claude Code / Cursor） |
| 依赖 | 零外部依赖——仅用标准库 |
| 存储 | 本地 Markdown 文件（无数据库、无 API） |
| 脚本 | 4 个 Python 文件：`env_detect.py`、`vfm_verify.py`、`eviction_score.py`、`bias_audit.py` |
| 参考文档 | 4 个：`record-templates.md`、`weekly-reflection.md`、`file-structure.md`、`claude-code-compat.md` |
| 总体积 | ~45KB（不含 banner） |
| 调度 | `openclaw cron` 或系统 `crontab` |

---

### 🛠️ 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/muippt/mu-self-evolve.git

# 2. 初始化（自动检测环境）
cd mu-self-evolve
python3 scripts/env_detect.py --init

# 3. 设置定时进化
openclaw cron add --name "🧬 每日自我进化" --cron "30 19 * * *" --message "执行每日自我进化流程"
```

就这样。Agent 会开始每日沉淀经验、每周反思进化。

---

### 🔒 安全与隐私

- **100% 本地运行** — 所有数据留在工作区的 `.learnings/` 目录，无云端、无 API 调用、无遥测。
- **零外部依赖** — Python 脚本仅使用标准库。
- **不动模型参数** — 通过组织 Markdown 文本工作，不做微调或权重更新。
- **用户可控淘汰** — 每次归档决策都有日志；偏差审计确保透明度。

---

### ⭐ Star 趋势

如果 mu-self-evolve 帮助你的 Agent 变得更聪明，欢迎给个 Star ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=muippt/mu-self-evolve&type=Date)](https://star-history.com/#muippt/mu-self-evolve&Date)

> 一个会从自己的错误中学习的 AI Agent — 无需微调。

---

### 👤 作者简介

🎓 清华大学出版社签约作家 / 2026当当影响力作家 / 某互联网大厂 AI 大模型业务 HR 砖家 / 一级人力资源管理师 / 二级心理咨询师 / 野生设计师

📚 著有[《图解团队管理》](https://item.m.jd.com/product/14547345.html)，服务客户有字节跳动、腾讯、百度、中国移动、SMG、BOE…

💡 [微信公众号](https://mp.weixin.qq.com/s/v1JSZvlN5fvbOOHvkvXEtA) / [小红书](https://xhslink.com/m/ESxtgUNMdl)：muippt

---

### 📄 许可证与致谢

[MIT](LICENSE) © 2026 muippt

本项目借鉴了以下研究的洞察：

- **ExpeL**（arXiv:2308.10144）— 经验提取双层检索（案例+规则）
- **Reflexion**（arXiv:2303.11366）— 语言化反思作为「语义梯度」
- **Voyager**（arXiv:2305.16291）— 可执行代码技能库 + 自动课程
- **MemGPT**（arXiv:2310.08560）— OS 式虚拟内存管理
- **Generative Agents**（arXiv:2304.03442）— 观察→反思→规划三层记忆
- **GSME**（2025）— 提议与授信分离（LLM诊断+确定性验证）
- **The Blind Curator**（2025）— 评委偏差与淘汰机制静默失效

行业参考：CrewAI（半衰期衰减）、Mem0（LLM-as-editor）、ChatGPT Dreaming（异步合成）、Zep（双时间线）。

> 声明：本项目大部分内容由 AI 辅助完成。如您认为您的作品被使用但未获得适当署名，请提交 issue。
