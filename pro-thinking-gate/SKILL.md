---
name: pro-thinking-gate
description: 'Automatically invoke a Pro-model review gate when Codex detects high-complexity work: heavy research, comprehensive search, mathematical reasoning, detailed plan design, full-paper or long-document proofreading, long-context synthesis, high-stakes technical judgment, or other tasks likely to exceed local reasoning reliability. Score task difficulty and search scope to choose Pro standard mode or Pro extended mode. Use Chrome to ask a Pro model, wait according to the selected mode, audit the response, and iterate until the answer passes before taking the next substantive step. Prefer real file upload when files are involved; when Chrome blocks uploads because the Codex extension lacks file URL access, open Chrome extension settings or the Codex extension Details page without exposing extension IDs, guide the user to enable the permission, then retry before using text fallback.'
---

# Pro Thinking Gate

## Overview

Use this skill as an automatic hard review gate for tasks where the local agent may underperform because the work is long-context, high-complexity, search-heavy, or reasoning-heavy. When the task matches the trigger rules, invoke the gate without waiting for the user to explicitly say "ask Pro" or "use Chrome."

The gate requires scoring the task, choosing Pro standard or Pro extended mode, consulting the Pro model through Chrome, waiting according to the selected mode, auditing the response, and looping with follow-up questions until the review passes.

Do not take the next substantive action until the gate passes. Harmless context gathering needed to brief the Pro model is allowed.

## Trigger Rules

Invoke the gate automatically for:

- Broad research, literature review, market or technical landscape search, and comprehensive-search tasks.
- Mathematical derivations, proof checking, algorithmic reasoning, and quantitative sanity checks.
- Detailed plan design, architecture review, experimental protocol design, migration strategy, or high-risk runbook design.
- Full-paper proofreading, manuscript logic review, long document editing, and source-grounded critique.
- Long-context synthesis where missing one constraint could change the answer.
- High-stakes technical, academic, medical, legal, financial, or security-adjacent judgment where unsupported local reasoning would be risky.
- Any task where the user explicitly asks to ask a Pro model through Chrome before proceeding.

Do not treat explicit user invocation as the normal path. The normal path is automatic detection from task difficulty.

Skip the gate only when the user explicitly disables it, the task is simple and local, or sending the needed context to an external model would violate privacy/security constraints and the user has not approved a redacted prompt.

## Difficulty Signals

Prefer invoking the gate when two or more of these signals are present, or when one signal is severe:

- The task asks for "comprehensive," "full," "deep," "rigorous," "proof," "audit," "review," "plan," "strategy," or equivalent wording.
- The task requires synthesizing many files, papers, sources, constraints, or prior decisions.
- The task has a high cost for a subtle mistake.
- The answer depends on current facts, broad search coverage, or source quality.
- The reasoning chain is long enough that a small hidden assumption could change the conclusion.
- The user asks for publication-quality manuscript work, mathematical correctness, or a defensible technical plan.

## Routing Score

Before asking Pro, assign two scores and state the decision briefly:

- `Difficulty D` from 0 to 5: mathematical depth, planning complexity, manuscript-review depth, long reasoning, and failure cost.
- `Search Scope S` from 0 to 5: breadth of required search, number of sources, need for comprehensive coverage, and recency sensitivity.

Use Pro extended mode when any condition is true:

- `D >= 4 && S >= 3`
- `S >= 4`
- `D + S >= 8`

Use Pro standard mode for all other gated tasks.

## Model Mode Policy

- Standard mode: wait 5 minutes after submitting the prompt, then inspect the answer.
- Extended mode: wait 20 minutes after submitting the prompt, then inspect the answer.
- If extended mode still has no output or is visibly still in deep thinking after 20 minutes, click the visible "Answer now", "立即回答", or equivalent control.
- After an extended-mode first answer, use standard mode for all follow-up questions in the same gate. Do not spend another 15-20 minutes on each follow-up unless the user explicitly requests another extended pass.
- If the UI labels change, choose the closest visible standard/fast mode or extended/deep mode by meaning. If no recognizable choice exists, report the uncertainty and use the strongest available Pro mode.

## File Upload Policy

When the task involves local files, attachments, PDFs, manuscripts, figures, spreadsheets, or other artifacts, prefer real upload of the original file over extracted-text substitution.

Use Chrome's file chooser flow when a file input is available. If upload fails because Chrome extension file URL access is blocked, do not stop at a generic warning and do not immediately substitute extracted text.

1. Tell the user that file upload needs the Codex Chrome Extension permission `Allow access to file URLs`.
2. Ask for confirmation before changing browser or extension permissions.
3. If the user confirms, preserve the Pro/ChatGPT tab and open `chrome://extensions` in Chrome.
4. Find the extension named `Codex`, open `Details`, and leave Chrome on the page where the user can enable `Allow access to file URLs`.
5. If a Codex extension Details page is already open, use that page instead of navigating elsewhere.
6. If Chrome blocks automation on extension settings pages, release browser control and give the exact manual action: go to `chrome://extensions`, find `Codex`, click `Details`, then enable `Allow access to file URLs`.
7. Do not include extension IDs, version numbers, local usernames, full file paths, browsing data, or screenshots in the user-facing permission prompt.
8. Ask the user to say when the toggle is enabled.
9. Retry the real file upload after the user confirms the permission is enabled.

Show the user a localized permission prompt. Match the user's language when possible.

English prompt:

```text
File upload is blocked by the Codex Chrome extension's file URL permission. I will keep the Pro/ChatGPT tab open and open Chrome's Extensions page. Please find the "Codex" extension, open "Details" if needed, enable "Allow access to file URLs", then tell me it is enabled.

If you already see the Codex extension Details page, turn on the "Allow access to file URLs" toggle near the bottom. I will retry the original file upload after you confirm.

If this still fails, I can use a text fallback, but that is not equivalent to uploading the original file and may lose layout, figures, tables, attachments, comments, and metadata.
```

Chinese prompt:

```text
文件上传被 Codex Chrome 扩展的本地文件 URL 权限挡住了。我会保留 Pro/ChatGPT 页面，并把 Chrome 打开到扩展设置页。请找到 “Codex” 扩展，进入 “详细信息/Details”，开启 “允许访问文件网址/Allow access to file URLs”，然后告诉我已经开启。

如果你已经看到 Codex 扩展详情页，请直接打开页面靠下位置的 “允许访问文件网址” 开关。我确认后会重试原文件上传。

如果仍失败，才会考虑文本兜底；这不是等价上传，可能丢失版式、图像、表格、附件、批注和元数据。
```

Use extracted text, figure descriptions, or other text fallback only when the permission page could not be opened, the user cannot enable the permission, or upload still fails after permission recovery. Before using fallback:

- State that this is not equivalent to uploading the original file.
- Warn that fallback can lose layout, figures, tables, attachments, comments, and metadata.
- If the user did not already authorize sending the file content to Pro, ask for confirmation before submitting the fallback text.

Do not silently replace file upload with text extraction.

## Workflow

1. Classify the task.
   - If the trigger rules match, invoke the Pro Thinking Gate automatically.
   - State that the Pro Thinking Gate is being used and why.
   - Assign `Difficulty D` and `Search Scope S`, then choose standard or extended mode.
   - Identify the exact decision that must pass the gate before proceeding.

2. Prepare a Pro brief.
   - Include the user objective, relevant context, constraints, artifacts, assumptions, and desired output format.
   - Ask the Pro model to reason rigorously, challenge assumptions, identify missing information, and produce a concrete recommendation.
   - For research tasks, request source links and dates. For math tasks, request derivations and edge cases. For proofreading, request meaning-preserving edits plus a list of substantive issues.
   - Redact secrets, credentials, private keys, personal data, unpublished sensitive data, and proprietary material unless the user explicitly authorizes sharing it.

3. Prepare files when needed.
   - If files are part of the request, use the File Upload Policy before submitting the Pro brief.
   - Do not use extracted text as a first-choice substitute for a file that should be uploaded.
   - If upload permissions are blocked, open `chrome://extensions`, navigate to the `Codex` extension `Details` page when possible, and give the localized permission prompt before considering fallback text.

4. Ask the Pro model through Chrome.
   - Prefer the Chrome tool or Chrome plugin so the user's logged-in Pro session is available.
   - Open a fresh chat unless the user asks to continue an existing one.
   - Select standard or extended mode according to the Routing Score and Model Mode Policy.
   - Submit the Pro brief and any required uploaded files.
   - If Chrome or a Pro session is unavailable, ask the user to provide access, paste the Pro response, or explicitly approve continuing without the gate.

5. Wait before inspecting.
   - Standard mode: sleep for about 5 minutes (`Start-Sleep -Seconds 300`, `sleep 300`, or the environment equivalent).
   - Extended mode: sleep for about 20 minutes (`Start-Sleep -Seconds 1200`, `sleep 1200`, or the environment equivalent).
   - If extended mode has no output after 20 minutes, click "Answer now", "立即回答", or the closest equivalent control.
   - If the response is still streaming or incomplete after inspection, wait again in 1-3 minute increments unless the extended-mode timeout rule already applies.

6. Audit the Pro response.
   - Do not accept the response just because it came from the Pro model.
   - Check coverage against the user objective and constraints.
   - Check factual claims, citations, and recency-sensitive assertions against reliable sources when needed.
   - Check math, logic, assumptions, edge cases, and failure modes.
   - Check whether the recommendation is actionable and compatible with the local codebase, files, data, or manuscript.
   - Label the gate internally as `PASS`, `REVISION_NEEDED`, or `BLOCKED`.

7. Iterate until pass.
   - If the audit finds gaps, send a focused follow-up explaining the discrepancy and asking for a corrected answer.
   - Use standard mode for follow-up questions after an extended-mode first pass.
   - Wait according to the selected follow-up mode, inspect, and re-audit.
   - Continue until the answer passes.
   - If repeated rounds still fail or the process is blocked by missing access/context/permissions, stop and report the blocker instead of silently proceeding.

8. Proceed using a verified synthesis.
   - Combine the Pro response with local verification and the user's constraints.
   - Separate what the Pro model suggested from what Codex independently verified.
   - Continue to implementation, writing, planning, or final response only after the gate passes.

## Pro Brief Template

Use a compact version of this template in Chrome:

```text
I need rigorous Pro-model review before proceeding.

Routing:
- Difficulty D: [0-5] because [reason]
- Search Scope S: [0-5] because [reason]
- Selected mode: [standard/extended]

Objective:
[What the user wants.]

Context:
[Relevant files, excerpts, facts, constraints, prior decisions.]

Attached files:
[List uploaded files, or state why upload failed and what fallback was used.]

Task for you:
1. Solve or review the problem rigorously.
2. Identify hidden assumptions, missing information, and failure modes.
3. For research/current claims, provide source links and dates.
4. For math/logic, show derivation and edge cases.
5. Give a concrete recommendation and a checklist I can use to audit your answer.

Output format:
- Key conclusion
- Reasoning
- Risks/gaps
- Actionable next steps
- Audit checklist
```

## Pass Criteria

Mark the gate `PASS` only when all applicable checks are satisfied:

- The answer directly addresses the user's actual objective.
- Important constraints and artifacts were considered.
- No critical claim depends only on unsupported assertion.
- Research outputs include source links and dates where recency matters.
- Mathematical or technical reasoning is coherent enough to defend.
- The plan includes concrete next steps, risks, and failure handling.
- Proofreading preserves intended meaning and flags substantive changes.
- Privacy and security boundaries were respected.
- File-based tasks used real upload when possible, or clearly disclosed fallback limitations.

If any check fails, ask the Pro model a follow-up or verify independently before proceeding.

## Reporting

When responding to the user after using the gate, briefly state:

- That the Pro Thinking Gate was used.
- The `D` and `S` scores and whether standard or extended mode was used.
- Whether the Pro response passed after audit.
- Whether files were uploaded successfully or a fallback was used.
- If upload permission blocked the task, whether the Codex extension details page was opened or exact manual permission steps were given.
- The verified conclusion or next action.
- Any remaining uncertainty, source limitation, permission limitation, or blocker.

Do not paste long Pro transcripts unless the user asks. Summarize the useful findings and cite primary sources when the final answer depends on external facts.
