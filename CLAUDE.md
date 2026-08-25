# Project Instructions

## Mandatory Code Quality Workflow

These rules apply to ANY code generation or edit in this repo (new files, edits, refactors). They override default behavior — do not skip them, even for small changes.

Two SonarQube MCP servers are registered in this environment: `sonarqube` and `sonarqube-mcp-server-sqc`. Tool names differ by server — always use the exact prefixed tool name below, do not guess the server.

### 1. Before writing code: fetch product guidelines
Before generating or editing any source code, call `mcp__sonarqube-mcp-server-sqc__get_guidelines` (mode="combined") with the relevant `categories`, `languages`, and `file_paths` for the task. Use the returned guidelines to inform the implementation.
Note: `get_guidelines` exists ONLY on `sonarqube-mcp-server-sqc` — the `sonarqube` server does not expose this tool.

### 2. After writing code: verify with SonarQube (both analyses, always)
After generating or editing any source code, run BOTH of the following on every changed/created file — never just one:
1. `mcp__sonarqube-mcp-server-sqc__run_advanced_code_analysis` (correct `fileScope`: MAIN or TEST, and current `branch`) — server-side, project-aware, catches cross-file issues.
2. Snippet analysis via the `sonarqube:analyze` (or `sonar-analyze`) skill/CLI — local, no project context, fast single-file check.

If either returns issues, fix them and re-run both until both come back clean before considering the task done.

No code change is complete until step 1, step 2 of this section, and step 1 (guidelines) above have all run.

<!-- sonar:begin:sonarqube-agentic-analysis-protocol -->
# Vortex analysis protocol

Vortex analysis is the final confirmation layer at the end of every turn in which you wrote to one or more files in the workspace (create, edit, patch, format — any tool call that changed file contents on disk).

**Trigger:** the moment before you send your final reply for the turn.

When the trigger fires, you **MUST** run Vortex analysis on the full set of files you modified this turn — **one command, not a per-file loop**.

**Preferred:** pass every path you modified this turn in a single invocation (repeat `--file` for each path). Use paths **relative to the project root** (e.g. `src/foo.ts`). For end-of-turn analysis, **always** pass `--depth DEEP` — even when you changed only one file — so cross-file findings are included.

```bash
sonar analyze agentic --project e-corp-demo_six_breakout_claude --depth DEEP --file <path/to/file1> --file <path/to/file2>
```

**Fallback:** when you cannot reliably list every modified path (e.g. you did not track them, or git state is the only source of truth), use the git change set with DEEP analysis:

```bash
sonar analyze agentic --project e-corp-demo_six_breakout_claude --depth DEEP
```

Per-edit hooks run faster STANDARD analysis. End-of-turn analysis must always use `--depth DEEP` (including a single `--file`). Use `--depth STANDARD` only for optional faster passes — never for the mandatory end-of-turn run.

Non-negotiable rules:

1. Mid-turn edits do **not** need Vortex analysis — only the final on-disk state matters. Skipping Vortex analysis at end-of-turn is a protocol violation.
2. Run Vortex analysis **before** sending your final reply, marking the task done, or handing control back to the user.
3. If Vortex analysis reports issues on lines you touched in this turn, fix them, then re-run Vortex analysis on the same scope (change set or explicit file list). Repeat until clean (or only pre-existing findings on lines you did not touch remain). Pre-existing findings on untouched lines are out of scope — do not "fix" them unless the user asked.
4. If Vortex analysis is skipped (no SonarQube Cloud connection, or no project configured), state the skip reason to the user once and continue — do not retry.
5. Do not suppress, summarize away, or omit Vortex analysis findings from your reply. Surface them verbatim.
<!-- sonar:end:sonarqube-agentic-analysis-protocol -->
