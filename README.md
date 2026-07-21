<p align="center">
  <img alt="mu-self-evolve" src="assets/default-banner.png" width="100%">
</p>

# mu-self-evolve

> A continuous self-evolution system for AI Agents — daily experience sedimentation + weekly error reflection, turning scattered conversations into structured persistent memory.

**English** | [中文](README_CN.md) | [🌐 Landing Page](https://muippt.github.io/mu-self-evolve/)

[![WeChat](https://img.shields.io/badge/muippt-07C160?logo=wechat&logoColor=white)](https://mp.weixin.qq.com/s/v1JSZvlN5fvbOOHvkvXEtA)
[![Xiaohongshu](https://img.shields.io/badge/muippt-FF2442?logo=xiaohongshu&logoColor=white)](https://xhslink.com/m/ESxtgUNMdl)
[![Book](https://img.shields.io/badge/Book-Visual%20Team%20Management-BBDDE5?logo=bookstack&logoColor=white)](https://item.m.jd.com/product/14547345.html)
[![License](https://img.shields.io/github/license/muippt/mu-self-evolve)](LICENSE)
[![Version](https://img.shields.io/github/v/release/muippt/mu-self-evolve)](https://github.com/muippt/mu-self-evolve/releases)
[![Stars](https://img.shields.io/github/stars/muippt/mu-self-evolve)](https://github.com/muippt/mu-self-evolve/stargazers)

### 💡 Usage Examples

🧠 **Daily Experience Sedimentation** — Automatically extract corrections, errors, and positive signals from today's conversations and log them into structured files.

🔍 **VFM Verification** — Before promoting any rule to permanent memory, run deterministic verification scripts that check recurrence count, cross-task scope, and resolution rate.

🪶 **Score-Based Eviction** — Memory entries decay over time with a 30-day half-life formula; low-value entries are archived automatically while high-recurrence ones are exempt.

🏷️ **WHERE×WHY Pathology Archiving** — Every error is tagged with WHERE (error stage) and WHY (root cause type), enabling structured pattern analysis instead of vague narratives.

🧪 **Bias Audit** — Inject synthetic test entries with known properties to verify the eviction mechanism hasn't silently degraded — preventing "The Blind Curator" problem.

🛠️ **Active Skill Synthesis** — When ≥3 entries in the same domain cluster together, automatically generate a Skill draft (SKILL.md skeleton + script framework) for the human to review.

⏰ **Scheduled Automation** — Set up daily cron jobs that run the full 6-step evolution workflow, with an extended weekly reflection every Friday.

📊 **Cross-Environment Support** — Auto-detects OpenClaw, Claude Code, or other LLM agent environments, adapting paths and scheduling automatically.

---

### ✨ Core Highlights

#### VFM Deterministic Verification

Traditional approaches rely on LLM self-assessment to decide which experiences deserve promotion. This is unreliable — LLMs can hallucinate confidence. mu-self-evolve implements a **propose-and-verify separation** (inspired by GSME, 2025):

| Step | Who | What |
|------|-----|------|
| Propose | LLM | Identifies a pattern and proposes a rule |
| Verify | Script (`vfm_verify.py`) | Counts recurrence, cross-task scope, resolution rate, pathology concentration → outputs confidence score 0–100 |
| Promote | Gate | Confidence ≥70 + Recurrence ≥2 + VFM ≥50 → promote to permanent memory |

#### Score + Decay Eviction

Replaces line-count hard truncation with a time-decay scoring model (inspired by CrewAI's half-life decay):

```
effective_score = VFM × 0.5^(days_since_last_seen / 30)
```

| State | Condition | Action |
|-------|-----------|--------|
| Promoted | status=promoted | Move to permanent memory |
| Resolved + old | status=resolved AND >30 days | Archive |
| High recurrence | Recurrence ≥2 | **Exempt** (overrides score and age) |
| Single + old | Recurrence=1 AND >14 days | Mark dormant, archive |
| Low score + old | effective<25 AND >14 days | Archive |
| Normal | effective≥25 OR <14 days | Keep |

#### WHERE×WHY Pathology Keys

Every error and learning entry is tagged with a structured pathology key:

**WHERE tags** (error stage): `skill_load` · `mcp_call` · `file_write` · `api_call` · `browser_op` · `memory_op` · `agent_dispatch` · `prompt_parse` · `auth` · `other`

**WHY tags** (root cause): `param_missing` · `token_expired` · `timing_race` · `perm_denied` · `knowledge_gap` · `format_mismatch` · `timeout` · `logic_error` · `config_drift` · `user_correction` · `other`

Both tag sets support user customization — add your own domain-specific tags as your agent encounters new environments.

#### Bias Audit (The Blind Curator Defense)

Inspired by "The Blind Curator" (2025) — if the eviction mechanism itself has a bias, it will silently kill valuable entries while the operator believes the system is working correctly. The `bias_audit.py` script:

1. Creates 5 synthetic test entries with known properties (should-keep, should-archive, should-override, etc.)
2. Runs them through the actual eviction logic
3. Compares actual results against expected results
4. Reports PASS/FAIL — any FAIL means the eviction logic has a bug

---

### 📌 Comparison

| Feature | mu-self-evolve | Raw Memory Files | Mem0 | Zep |
|---------|---------------|-------------------|------|-----|
| Daily auto-sedimentation | ✅ 6-step workflow | ❌ Manual | ❌ API-only | ❌ API-only |
| Deterministic verification | ✅ Script-based | ❌ | ✅ LLM-as-editor | ❌ |
| Time-decay eviction | ✅ 30-day half-life | ❌ | ✅ Half-life | ✌️ Dual-timeline |
| Pathology archiving (WHERE×WHY) | ✅ Structured tags | ❌ | ❌ | ❌ |
| Bias audit | ✅ Defect injection | ❌ | ❌ | ❌ |
| Active Skill synthesis | ✅ Cluster → draft | ❌ | ❌ | ✌️ Dreaming |
| Cross-environment support | ✅ OpenClaw + Claude Code | N/A | ❌ | ❌ |
| No external API required | ✅ Fully local | ✅ | ❌ | ❌ |

---

### 🚀 Workflows

| Workflow | Scenario | Trigger |
|----------|----------|---------|
| Daily Evolution (Steps 1-6) | Extract today's experiences, verify, promote, summarize | Scheduled cron (daily) |
| Weekly Reflection (Steps 6a-6g) | Error scanning, narrative generation, eviction, Skill synthesis | Friday (merged with daily) |
| On-Demand Recording | User corrects agent or error occurs | Trigger detection (real-time) |
| Manual Verification | Check a specific rule's confidence | `python3 vfm_verify.py <pattern-key>` |
| Eviction Audit | Run bias test + score-based cleanup | `python3 bias_audit.py && python3 eviction_score.py` |

---

### ⚙️ Technical Specs

| Item | Description |
|------|-------------|
| Runtime | Python 3.8+ (scripts only); any LLM Agent (OpenClaw / Claude Code / Cursor) |
| Dependencies | Zero external packages — standard library only |
| Storage | Local markdown files (no database, no API) |
| Scripts | 4 Python files: `env_detect.py`, `vfm_verify.py`, `eviction_score.py`, `bias_audit.py` |
| References | 4 docs: `record-templates.md`, `weekly-reflection.md`, `file-structure.md`, `claude-code-compat.md` |
| Total Size | ~45KB (excluding banner) |
| Scheduling | `openclaw cron` or system `crontab` |

---

### 🛠️ Quick Start

```bash
# 1. Clone
git clone https://github.com/muippt/mu-self-evolve.git

# 2. Initialize (auto-detects environment)
cd mu-self-evolve
python3 scripts/env_detect.py --init

# 3. Schedule daily evolution
openclaw cron add --name "🧬 Daily Evolution" --cron "30 19 * * *" --message "Run mu-self-evolve daily workflow"
```

That's it. The agent will start sedimenting experiences daily and reflecting weekly.

---

### 🔒 Security & Privacy

- **100% Local** — All data stays in your workspace's `.learnings/` directory. No cloud, no API calls, no telemetry.
- **No External Dependencies** — Python scripts use only the standard library.
- **No Model Parameters** — Works by organizing text in markdown files, not by fine-tuning or updating weights.
- **User-Controlled Eviction** — Every archive decision is logged; the bias audit ensures transparency.

---

### ⭐ Star History

If mu-self-evolve helps your agent get smarter, consider giving it a star ⭐

[![Star History Chart](https://api.star-history.com/svg?repos=muippt/mu-self-evolve&type=Date)](https://star-history.com/#muippt/mu-self-evolve&Date)

> An AI Agent that learns from its own mistakes — no fine-tuning required.

---

### 👤 About the Author

🎓 Signatory Author of Tsinghua University Press / 2026 Dangdang Influential Author / AI & Large Model Business HR Specialist at a Leading Tech Company / National Level-1 HR Manager / Level-2 Psychological Counselor / Self-taught Designer

📚 Author of [*Visual Team Management*](https://item.m.jd.com/product/14547345.html). Clients include ByteDance, Tencent, Baidu, China Mobile, SMG, BOE…

💡 [WeChat Official Account](https://mp.weixin.qq.com/s/v1JSZvlN5fvbOOHvkvXEtA) / [Xiaohongshu](https://xhslink.com/m/ESxtgUNMdl): muippt

---

### 📄 License & Acknowledgments

[MIT](LICENSE) © 2026 muippt

This project builds on insights from the following research:

- **ExpeL** (arXiv:2308.10144) — Experience extraction with dual-layer retrieval
- **Reflexion** (arXiv:2303.11366) — Linguistic reflection as "semantic gradient"
- **Voyager** (arXiv:2305.16291) — Executable skill library + automatic curriculum
- **MemGPT** (arXiv:2310.08560) — OS-style virtual memory management
- **Generative Agents** (arXiv:2304.03442) — Observation → reflection → planning memory hierarchy
- **GSME** (2025) — Propose-and-verify separation (LLM diagnosis + deterministic verification)
- **The Blind Curator** (2025) — Curator bias and silent eviction mechanism failure

Industry references: CrewAI (half-life decay), Mem0 (LLM-as-editor), ChatGPT Dreaming (async synthesis), Zep (dual-timeline).

> Note: Much of this project was co-created with AI assistance. If you believe your work has been used without proper attribution, please open an issue.
