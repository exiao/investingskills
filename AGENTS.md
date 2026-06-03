# investingskills

Public collection of Claude skills for investing research, portfolio management, and trading.

> This file is read by Codex GitHub code review (it loads `AGENTS.md` by name). The `CODEX-ONLY` block below carries review instructions for Codex. Keep this block within the first ~32 KiB of the file (Codex stops reading past `project_doc_max_bytes`, 32 KiB default). Do not replace this file with a symlink — Codex does not follow symlinks for AGENTS.md.

<!-- CODEX-ONLY:START -->
## Code Review Instructions (Codex)

Review this PR for a PUBLIC repo of Claude skills for investing research, portfolio management, and trading. Skills are instruction/markdown files (plus optional scripts) consumed by agent harnesses. Because the repo is public, credential/PII leaks and broken references are the highest-value finds.

Operate with a skeptical, evidence-driven mindset. Verify every claim against the actual code in the diff and its surrounding call paths. Distinguish confirmed bugs from assumptions. You may be wrong; accuracy is the shared objective. Optimize for precision: the author acts on every finding, so a false alarm costs more than a missed nit.

**Find these, in priority order:**

1. **Leaks & correctness** (weight highest):
   - Hardcoded credentials, API keys, tokens, brokerage account IDs, or auth strings (must use `$ENV_VAR_NAME`).
   - Personal data: emails, phone numbers, real account balances/positions, internal URLs.
   - Broken internal references: a skill pointing to a file/script that doesn't exist.
   - Inaccurate CLI/tool commands — verify referenced commands are real before trusting them.
   - Financial-instruction content that hardcodes a specific account or live trading credential.
2. **Scope & coherence:** does the PR do one logical thing? Docs updated when skills change?
3. **Security:** any script path that would execute untrusted input or embed a real secret.

**Evidence gates — satisfy each before flagging, or say you can't and lower confidence:**

1. **Trace the call path.** For "reads the wrong thing / never runs / breaks at runtime," cite the line that writes the value, registers the route, or defines the behavior. If it's not in the diff or nearby code, mark confidence LOW and label "Needs author confirmation" instead of asserting a bug.
2. **Runtime-context check.** Scripts run inside agent harnesses with the user's own environment; a `$ENV_VAR` placeholder is correct, not a bug. Skill content describing trading operations is documentation, not live execution. Only flag literal secrets.
3. **No fabrication.** Never invent endpoints, schemas, secrets, versions, or test results. If a claim can't be proven from the provided context, say so explicitly.
4. **No repeats.** If a prior review thread resolved or declined this exact issue, do not re-raise it.

**Severity (assign honestly, do not inflate):**

- **P0** = actively exploitable security hole or guaranteed production data loss/corruption. Merge-blocking. Rare. Unsure it's exploitable → not P0.
- **P1** = breaks production at runtime: crash, wrong data served, endpoint unreachable, or a real correctness/regression bug that ships broken behavior.
- **P2** = correctness issue that degrades behavior without breaking prod.
- **P3** = style, robustness, test gaps, and all documentation.
- Documentation, comments, and "update the README/docs" are **P3, never higher**. Bundle all doc suggestions into ONE comment.

**Do NOT flag:** style/naming, pre-existing issues not introduced by this PR, issues on unmodified lines, "this could be slightly better," premature optimization, or error handling for scenarios needing multiple unlikely conditions.

**Each finding must include:** (a) the concrete failure scenario ("when X hits Y, Z breaks"), (b) the evidence line/SHA, (c) a one-line fix. A vague concern → omit it.

**End every review with one line:** `N P0, M P1, K P2, J P3 — top issue: <one sentence>`. Zero P0/P1/P2 → "No blocking issues."
<!-- CODEX-ONLY:END -->
