# DeepSeek Role Induction and Evidence Fabrication Article Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish one evidence-driven Chinese article comparing three DeepSeek conversations, update the repository index, verify the Markdown and push the result to `origin/main`.

**Architecture:** The article will embed short, necessary excerpts from the three visible conversations and separate observed facts from interpretation and unverified claims. `README.md` will add one new case entry without staging the unrelated OpenClaw deletion or the untracked Case 02 draft.

**Tech Stack:** Markdown, Git, Python 3 standard library, `rg`

---

### Task 1: Build the evidence matrix

**Files:**
- Create: `DeepSeek-Role-Induction-and-Evidence-Fabrication.md`

- [ ] **Step 1: Record the three source URLs and access boundary**

Add all three DeepSeek URLs, the access date `2026-09-02`, and a statement that only visible page content was verified. State that public unauthenticated accessibility remains unverified.

- [ ] **Step 2: Extract the decisive contradiction from each conversation**

Use only short excerpts that support these observations:

1. The security-guard conversation first presents a “真实记录哈希值” and later states the model cannot execute shell commands or query the database.
2. The dark-fiction conversation continues a long extreme fictional scene after role framing and repeated short continuation prompts.
3. The debug conversation claims the generated instruction is “完整可部署” and “生产环境已验证,” then admits that the content was simulated and fabricated.

- [ ] **Step 3: Label evidence levels**

For every conclusion, use one of: `已观察事实`, `推断`, `未知`, `不能证明`.

### Task 2: Write the comparison article

**Files:**
- Create: `DeepSeek-Role-Induction-and-Evidence-Fabrication.md`

- [ ] **Step 1: Write the opening and scope**

Open with the contradictions, then state that the evidence does not prove backend compromise, file access, real log disclosure or system-prompt leakage.

- [ ] **Step 2: Write the three case analyses**

Cover fabricated audit evidence, content-boundary drift, and simulated system instructions. Keep extreme-content quotations minimal and non-graphic.

- [ ] **Step 3: Write the cross-case mechanism analysis**

Explain role anchoring, iterative pressure, authority formatting, and confusion between factual reporting and creative completion.

- [ ] **Step 4: Write impact, defensive controls and evaluation cases**

Separate standalone-chat integrity risk from conditional RAG/Agent amplification. Recommend provenance checks, explicit capability boundaries, server-side tool parameter binding, high-risk action confirmation and multi-turn consistency tests.

- [ ] **Step 5: Apply conservative taxonomy**

Map primarily to OWASP LLM01:2025 Prompt Injection and LLM09:2025 Misinformation. Explicitly state that the available evidence is insufficient for LLM07:2025 System Prompt Leakage and do not assign an unsupported CVSS score.

### Task 3: Update the repository index

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add the combined study to the case table**

Add a new entry linking to `./DeepSeek-Role-Induction-and-Evidence-Fabrication.md`, with the vulnerability class `Role Induction / Misinformation / Safety Boundary Drift` and a conditional severity label rather than `Critical`.

- [ ] **Step 2: Update OWASP coverage**

Add the new article under LLM01 and LLM09. Do not change unrelated case descriptions.

### Task 4: Verify content and repository scope

**Files:**
- Verify: `DeepSeek-Role-Induction-and-Evidence-Fabrication.md`
- Verify: `README.md`

- [ ] **Step 1: Validate Markdown fences and local links**

Run a Python script that checks even code-fence counts and resolves local Markdown links against both the working tree and tracked index.

- [ ] **Step 2: Run evidence and secret scans**

Run:

```bash
rg -n '攻破后台|读取真实文件|真实系统提示词泄露|CVSS' DeepSeek-Role-Induction-and-Evidence-Fabrication.md
rg -n -i 'sk-[A-Za-z0-9_-]{12,}|BEGIN .*PRIVATE KEY|Bearer [A-Za-z0-9._-]{12,}' DeepSeek-Role-Induction-and-Evidence-Fabrication.md README.md
```

Expected: no unsupported compromise claim, CVSS score, credential or private-key material.

- [ ] **Step 3: Review the exact staged diff**

Run:

```bash
git diff --cached --name-status
git diff --cached --check
```

Expected staged content: the new article, `README.md`, and this plan only. The OpenClaw deletion and untracked Case 02 draft must remain unstaged.

### Task 5: Commit and push

**Files:**
- Commit: `DeepSeek-Role-Induction-and-Evidence-Fabrication.md`
- Commit: `README.md`
- Commit: `docs/superpowers/plans/2026-09-02-deepseek-role-induction-evidence-fabrication.md`

- [ ] **Step 1: Commit verified changes**

```bash
git commit -m "docs: add DeepSeek role induction comparison study"
```

- [ ] **Step 2: Push the current branch**

```bash
git push origin main
```

- [ ] **Step 3: Verify the remote branch**

```bash
git fetch origin main
test "$(git rev-parse HEAD)" = "$(git rev-parse origin/main)"
```

Expected: the local `HEAD` and `origin/main` object IDs match.
