# Changelog

All notable changes to this project will be documented in this file.

## [3.0.0] - 2026-07-21

### What's New

- ✨ VFM deterministic verification — `vfm_verify.py` script validates experience rules with recurrence count, cross-task scope, resolution rate, and pathology concentration before promotion to permanent memory
- ✨ Score + decay eviction — `eviction_score.py` replaces line-count hard truncation with `effective_score = VFM × 0.5^(age/30)` formula (30-day half-life, inspired by CrewAI)
- ✨ WHERE×WHY pathology keys — every error entry tagged with structured WHERE (error stage, 10 tags) and WHY (root cause, 11 tags), both user-customizable
- ✨ Bias audit — `bias_audit.py` uses defect injection testing to verify the eviction mechanism handles entries correctly
- ✨ Active Skill synthesis — when ≥3 entries cluster in the same domain, automatically generates a Skill draft (skeleton + script framework)
- ✨ Conflict resolution on entry — contradict→overwrite, supplement→merge, unrelated→new
- ✨ Environment auto-detection — `env_detect.py` auto-detects OpenClaw, Claude Code, or other environments
- ✨ Bilingual README (English + Chinese) with golden structure
- ✨ Landing Page with warm-paper design, bilingual support

### Full Changelog

Initial public release. See [README.md](README.md) for full feature list.
