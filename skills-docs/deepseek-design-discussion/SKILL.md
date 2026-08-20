---
name: deepseek-design-discussion
description: >
  Discuss project design with DeepSeek through multi-round conversations.
  Use when the user wants to brainstorm, design, or refine a project architecture,
  discuss technical decisions, get a second opinion on system design,
  or iteratively develop a design document with DeepSeek.
  Triggers: "和 deepseek 讨论", "跟 deepseek 设计", "deepseek 讨论方案",
  "用 deepseek 讨论", "deepseek 设计", "讨论方案", "讨论设计"
---

# DeepSeek Design Discussion

Multi-round structured discussion with DeepSeek via Browser Relay. Drive the user's own Chrome to reuse DeepSeek login state — no white screen, no re-login.

## Prerequisites

1. **OMP Browser Relay** installed and enabled (see `omp://tools/browser.md`):
   ```bash
   omp browser-relay install
   omp config set browser.relay true
   ```
2. **User action**: Chrome extension loaded at `chrome://extensions` → Developer mode → Load unpacked → `C:\Users\JIE\.omp\browser-relay\extension`
3. DeepSeek tab opened via Relay before running discussion rounds.

## DeepSeek Interface Map

| Element | Selector / Ref | Notes |
|---------|---------------|-------|
| Input box | `[placeholder*="发送消息"]` | multiline textbox, sends on Enter |
| Send | `tab.press('Enter')` | |
| File upload | `<input type="file">` | accept: `.py` `.md` `.txt` `.json` `.csv` `.js` `.ts` `.html` `.pdf` + more |
| 快速模式 | radio `checked=true` | default, fast responses |
| 专家模式 | radio | deep thinking (R1-style reasoning) |
| 识图模式 | radio | image analysis mode |
| Response area | `tab.extract()` | use after each round |

## Workflow

### Step 1 — Open DeepSeek

```json
{"action":"open","app":{"relay":true},"url":"https://chat.deepseek.com/"}
```

Wait for `Opened tab "main" on relay`. If DeepSeek requires login, tell user to log in manually via their Chrome (the Relay drives their browser).

### Step 2 — Upload context file (optional but recommended)

If the user provided a file (code, doc, JSON schema):

```js
// run inside tab
const input = await tab.evaluate(() => document.querySelector('input[type="file"]'));
await tab.uploadFile(input, '/absolute/path/to/file.py');
```

Wait 1-2s for upload to register.

### Step 3 — Run discussion rounds

Use the **Discussion Frame** below for each round. Type message, press Enter, wait for reply, extract text.

```js
// Per-round pattern (run inside browser tool)
await tab.click('[placeholder*="发送消息"]');
await tab.type('[placeholder*="发送消息"]', '<MESSAGE>');
await tab.press('Enter');
await tab.waitFor(8000 + complexityBudget);  // adjust per round
const reply = await tab.extract();
```

### Step 4 — Multi-round loop

Execute rounds sequentially. Each round builds on prior consensus.

## Discussion Frame

### Round 1 — Context & Requirements

**Goal**: Align on what we're building and why.

**Prompt template**:
```
<project context — 2-3 sentences on what we're building>

Please help me design this. Before proposing anything:
1. What are the key requirements I haven't stated?
2. What are the top 3 design risks you see?
3. What 2-3 architecture options exist for the core problem?

Don't propose a full design yet — I want to align on problem framing first.
```

**After reply**: Summarize DeepSeek's framing back to user. Ask: *"DeepSeek says X as the core risk — do you agree? Which option do you lean toward?"*

### Round 2 — Architecture & Design

**Goal**: Settle on architecture direction.

**Prompt template**:
```
Based on your questions, here's what I'm thinking:

Requirements: <confirmed list>
Core risk: <agreed risk>
Leaning toward: <user's preferred option>

Given this:
1. What's the minimal viable architecture?
2. What are the key components and their responsibilities?
3. What's the data flow between them?
4. What should we build first?
```

**After reply**: *"DeepSeek proposes X architecture with Y components. Key decision: Z. Thoughts?"*

### Round 3 — Implementation Details

**Goal**: Drill into specifics.

**Prompt template**:
```
Architecture confirmed: <summary>

Now drill into <specific component / decision>:
1. <specific question 1>
2. <specific question 2>
3. <specific question 3>

Be concrete — suggest exact APIs, data structures, file layouts if possible.
```

### Round 4 — Review & Consensus

**Goal**: Confirm design, capture decisions, identify open questions.

**Prompt template**:
```
Let's lock in the design. Please produce:

1. **Architecture summary** — one-paragraph overview
2. **Components** — list with 1-line responsibility each
3. **Data flow** — how data moves between components
4. **Key decisions** — table: Decision | Choice | Rationale | Trade-off
5. **Open questions** — what still needs to be decided
6. **Suggested first step** — what to build first
```

### Round N — Ad hoc follow-up

For specific concerns (security, performance, edge cases):
```
On <topic>: <specific concern>

Context: <brief reminder of relevant design decisions>

1. What's the best practice here?
2. What are the concrete options?
3. What would you recommend for our case?
```

## Output: Design Document

After all rounds, synthesize into `design-discussion-summary.md`:

```markdown
# Design Discussion Summary

## Project
<one-line description>

## Architecture Overview
<final architecture paragraph>

## Components

| Component | Responsibility | Key Decisions |
|-----------|---------------|---------------|
| ... | ... | ... |

## Data Flow
<text or mermaid diagram>

## Key Decisions

| # | Decision | Choice | Rationale | Trade-off |
|---|----------|--------|-----------|-----------|
| 1 | ... | ... | ... | ... |

## Open Questions
- [ ] ...
- [ ] ...

## Action Items
- [ ] ...
```

## Extracting Long Replies

DeepSeek responses can be long. Use:

```js
const reply = await tab.extract();
```

If `extract()` truncates, use `tab.ariaSnapshot()` and parse specific sections.

## Switching Modes Mid-Discussion

- **专家模式** (deep thinking): before Round 2 or 3 for deeper reasoning
- **快速模式**: for follow-up questions
- **识图模式**: if user wants to discuss a diagram/image

Switch by clicking the radio button via `tab.click()`.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Relay not connecting | User must load extension at `chrome://extensions` |
| DeepSeek not logged in | Relay drives user's Chrome — tell them to log in manually |
| Input box not found | Run `tab.ariaSnapshot()` to find current ref |
| Reply truncated | Increase `waitFor()` or use `tab.ariaSnapshot()` |
| File upload fails | Check file extension is in accept list |

## Constraints

- **Never** open a headless browser for this — always use `app.relay: true`
- Always tell user what DeepSeek replied before asking next question
- Track consensus explicitly each round: *"Agreed on X, still deciding Y"*
- Never skip rounds — the frame is sequential for a reason
