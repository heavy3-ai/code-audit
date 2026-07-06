---
name: h3
description: Heavy3 Code Audit - Multi-model code review for coding agents via OpenRouter. Reviews uncommitted changes, staged files, the last commit, a commit range, a GitHub PR, or a plan document with a single model (DeepSeek V4 Pro) or a 3-model council (GPT 5.5 + Gemini 3.1 Pro + Grok 4), always showing a cost estimate for user confirmation before any API call. Use when the user asks for /h3, a Heavy3 review, an external or second-opinion model review, or a council review of a diff, PR, plan, or commit range. (Sponsored by Heavy3.ai)
argument-hint: "[pr <number>] [plan <file>] [<file>.md] [<range>] [--council] [--staged] [--commit] [--free] [--model MODEL]"
allowed-tools: Read, Bash, Glob, Grep
disable-model-invocation: true
---

# Heavy3 Code Audit - The Multi-Model Code Review for Coding Agents

**Sponsored by [Heavy3.ai](https://heavy3.ai) - Multi-model AI for high-stakes decisions**

You are helping the user get AI-powered code reviews via OpenRouter.

**All features are free and open source:**
- Single model review with DeepSeek V4 Pro (strong reasoning at low cost)
- 3-model council with GPT 5.5 + Gemini 3.1 Pro + Grok 4
- Per-model context budgets, 200K to 2M characters (see Context Limits)

**Web search:** reviews run WITHOUT web search. The switch is `enable_web_search` in config.json and it ships set to `false`. Do not advertise or rely on web search unless the user has turned that switch on.

## Arguments

`$ARGUMENTS` can contain:

**Explicit targets (no confirmation needed):**
- `pr <number>` - Review a GitHub pull request by number
- `plan <path>` - Review a specific plan file
- `<file>.md` - Shorthand for plan review (any .md file)
- `<range>` - Review a commit range (e.g., `HEAD~3..HEAD`, `abc123..def456`)

**Scope modifiers:**
- `--staged` - Force review of only staged changes
- `--commit` - Force review of the last commit only

**Mode options:**
- `--council` - Use 3-model council (GPT 5.5 + Gemini 3.1 Pro + Grok 4)
- `--free` - Use rotating free model from config
- `--model <name>` - Override model (shortcuts: glm, gpt, kimi, deepseek, free)

---

## Smart Detection

**When `/h3` is invoked without explicit targets, automatically detect intent and confirm with user.**

### Detection Priority

| Priority | Condition | Action |
|----------|-----------|--------|
| 1 | Explicit argument provided | Execute directly, no confirmation |
| 2 | Uncommitted changes exist | Confirm: review changes? |
| 3 | No changes + plan detected | Confirm: review the plan? |
| 4 | No changes + no plan | Ask: review commits or specify target? |

### Step-by-Step Smart Detection Workflow

**Step 1: Check for explicit arguments**

If `$ARGUMENTS` contains any explicit target or scope modifier (see Arguments), skip detection and execute directly via the matching workflow in Your Task.

**Step 2: Check for uncommitted changes**

Run: `git status --porcelain`

If output is NOT empty (changes exist):
```markdown
## Review Scope

I detected uncommitted changes:
- **Staged**: [X] files
- **Unstaged**: [Y] files

**Review all changes?** (y/n)
```

If user confirms, proceed with code review of all changes (`git diff HEAD`).

**Step 3: Check for plan (if no changes)**

Check these locations in order:
1. **Conversation context**: Did Claude just create or discuss a plan in this session?
2. **Current directory**: Does `plan.md`, `PLAN.md`, or `*.plan.md` exist?
3. **Plans folder**: Most recent `.md` file in `~/.claude/plans/`

If plan found:
```markdown
## Plan Detected

Found plan: `[path/to/plan.md]`
Last modified: [date]

**Review this plan?** (y/n)
```

If user confirms, proceed with plan review.

**Step 4: No changes and no plan - ask user**

```markdown
## No Changes Detected

No uncommitted changes or plans found.

**What would you like to review?**
1. Latest commit (`HEAD~1..HEAD`)
2. Recent commits (specify range, e.g., `HEAD~3..HEAD`)
3. Specific file or folder
4. Cancel
```

Wait for user response and proceed accordingly.

### Commit Range Support

For features/bug fixes spanning multiple commits, any `<start>..<end>` range works: `HEAD~1..HEAD` (last commit), `HEAD~3..HEAD` (last 3), `abc123..HEAD`, `abc123..def456`. The review diff is `git diff <range>`. When the user specifies a range, show a short commit summary table (hash, date, author, message, plus total +/- lines and file count) before the review; the Commit Range Review Workflow below has the exact commands.

## Configuration

Read the config from: `${CLAUDE_SKILL_DIR}/config.json`. Read the file for live values instead of assuming numbers; it is the single source of truth for models, budgets, and switches.

| Key | Meaning |
|-----|---------|
| `model` | Default single-review model (DeepSeek V4 Pro) |
| `free_model` | Model used with `--free` |
| `council_models` | The 3 council reviewers (correctness / performance / security) |
| `reasoning` | Reasoning effort passed to OpenRouter |
| `docs_folder` | Project docs folder to include in review context |
| `max_file_size` | Per-file character cap to respect when adding file contents to the context |
| `max_context` | Fallback context character budget (see Context Limits) |
| `max_context_by_model` | Per-model context character budgets (see Context Limits) |
| `enable_web_search` | Web search switch; ships `false`, so reviews run without web search |

API key is stored in `${CLAUDE_SKILL_DIR}/.env`:
```
OPENROUTER_API_KEY=your-key-here
```

## Preprocessed Context

### Git Status
!`git status --short 2>/dev/null || echo "Not a git repo"`

### Changed Files
!`git diff HEAD --name-only 2>/dev/null || echo "No changes"`

### Git Diff (truncated to 10000 chars)
!`git diff HEAD 2>/dev/null | head -c 10000 || echo "No diff"`

---

## Mode Routing

### IF ARGUMENTS contains "--council":
1. Run council.py instead of review.py
2. After council reviews, YOU synthesize findings with comparison table

### IF ARGUMENTS contains "--free":
1. Read free_model from config.json
2. Warn user: "Note: Free models rotate on OpenRouter. Your configured model may be unavailable."
3. If API call fails with model error:
   - Run: `python3 "${CLAUDE_SKILL_DIR}/scripts/list-free-models.py" --json`
   - This script needs the third-party `requests` package (`pip install requests`); review.py and council.py are stdlib-only
   - Expect a JSON list of free models; if the installed script version prints a human-readable table instead, read the model IDs from the table (same data either way)
   - Show available free models
   - Ask: "Pick a new free model?"
   - If user selects, UPDATE config.json with new free_model using Bash (e.g., `python3 -c "import json; ..."` to update the JSON file)
   - Retry with new model

---

## Scope Options

| Scope | Git Command | Use Case |
|-------|-------------|----------|
| Smart (default) | Auto-detected | Let `/h3` figure out what to review |
| `--staged` | `git diff --cached` | Force review of only staged changes |
| `--commit` | `git diff HEAD~1..HEAD` | Force review of the last commit |
| `<range>` | `git diff <range>` | Review multiple commits (e.g., `HEAD~3..HEAD`) |

**Error messages:**
- No staged changes (--staged): "No staged changes detected. Stage your changes first with `git add`."
- No commits (--commit): "No commits found. Make a commit first."
- Invalid range: "Invalid commit range. Check that both commits exist."

---

## Context Limits

There is no single fixed limit. review.py and council.py truncate the assembled context to a per-model CHARACTER budget:

1. `max_context_by_model` in config.json maps model IDs to character budgets (shipped values range from 200K to 2M chars depending on the model). Lookup order: exact model ID, then the ID with its `:online`/`:free` suffix stripped, then prefix match.
2. A model with no entry falls back to `max_context` (500K chars in the shipped config).
3. Anything past the budget is cut off by the script with a `[... truncated due to length ...]` marker.

Rule of thumb: 1 token ≈ 4 characters. When sizing a review, read the live budgets from `${CLAUDE_SKILL_DIR}/config.json` for the model you are about to use.

---

## Cost Estimation

**Before running a review**, estimate and display the cost to the user.

Pricing table, estimation formula, and worked examples live in `${CLAUDE_SKILL_DIR}/references/cost-estimation.md`. Read that file only when you reach this step (it is not needed for routing or argument parsing).

### Display Cost Estimate and Confirm

**IMPORTANT: Gather ALL context and save the temp JSON file FIRST, then calculate the cost estimate from the actual context size. This ensures an accurate estimate. The cost estimate is the ONLY user-facing prompt — do not interrupt the user for anything else.**

After gathering context and saving to the unique context file (`$H3_CONTEXT_FILE`), but BEFORE calling the review API:

```markdown
## Cost Estimate

| Metric | Value |
|--------|-------|
| Context size | ~[X]K chars |
| Est. input tokens | ~[X]K |
| Model(s) | [model name(s)] |
| **Est. cost** | **~$[X.XX]** |

**Proceed with review?** (y/n)
```

**Wait for user to confirm before submitting.**

If user declines, exit gracefully: "Review cancelled."

---

## Handle Large Changes First

**Before executing any review workflow**, check if changes are too large.

### Step 0: Estimate Context Size
- Count characters in: diff + file contents + docs + tests
- Check against the active model's character budget (see Context Limits)

**Quick size indicators (likely too large):**
- More than 50 changed files
- More than 10,000 additions + deletions
- Total characters exceed the model's budget

If any indicator trips, **DO NOT proceed automatically**. Read `${CLAUDE_SKILL_DIR}/references/large-changes.md` (read it only when this happens) and follow its module-selection and module-by-module review workflow, including the progress table and cross-module synthesis.

---

## Your Task

**Follow the Smart Detection workflow, then execute the appropriate review.**

### Step 0: Parse Arguments and Apply Smart Detection

1. **Check for explicit targets first** (skip detection if found):
   - `pr <number>` → Go to PR Review workflow
   - `plan <path>` or `<file>.md` → Go to Plan Review workflow
   - `<range>` (e.g., `HEAD~3..HEAD`) → Go to Commit Range Review workflow
   - `--staged` → Go to Code Review workflow (staged scope)
   - `--commit` → Go to Last Commit Review workflow

2. **If no explicit target, run Smart Detection** (see the Smart Detection section above) and route to the confirmed workflow.

### Shared Review Pipeline (every workflow ends with this)

After gathering the workflow-specific inputs listed below:

1. Read FULL content of each changed file
2. Find relevant documentation (CLAUDE.md, docs folder from config)
3. Include related test files (code, commit, range, and PR reviews; not plan reviews)
4. Find cross-file dependencies (see below; code and PR reviews only)
5. Include conversation context (see below)
6. **Generate a unique context file path and save context JSON using Bash** (see Temp File Handling — do NOT use the Write tool). Schema: see Context JSON Format.
7. **Calculate accurate cost estimate from the actual context size, display, and wait for user confirmation** (see Cost Estimation section)
8. If user confirms, run the review script immediately:
   - Default: `python3 "${CLAUDE_SKILL_DIR}/scripts/review.py" --type <code|plan|pr> --context-file "$H3_CONTEXT_FILE"`
   - With --council: `python3 "${CLAUDE_SKILL_DIR}/scripts/council.py" --type <code|plan|pr> --context-file "$H3_CONTEXT_FILE"`

### Code Review Workflow (uncommitted changes)

1. Determine scope based on arguments:
   - Default: `git diff HEAD` (all changes)
   - With `--staged`: `git diff --cached` (staged only)
2. Get full diff using appropriate git command
3. Get changed files list
4. Run the Shared Review Pipeline with `--type code`

### Last Commit Review Workflow (`--commit`)

1. Check if commits exist: `git log -1 --oneline 2>/dev/null`
2. If no commits, report: "No commits found. Make a commit first." and exit
3. Get last commit metadata: `git log -1 --pretty=format:"%H|%s|%an|%ad" --date=short` for hash, subject, author, date
4. Get the diff: `git diff HEAD~1..HEAD`
5. Get changed files: `git diff HEAD~1..HEAD --name-only`
6. Add the metadata to the context JSON under `commit_metadata` (see Context JSON Format)
7. Run the Shared Review Pipeline with `--type code`

### Commit Range Review Workflow (`<range>` like `HEAD~3..HEAD`)

1. Parse the range from arguments (e.g., `HEAD~3..HEAD`, `abc123..def456`)
2. Validate range: `git rev-parse <start> <end> 2>/dev/null`; if invalid, report error and exit
3. Show commit summary (informational): `git log --oneline --reverse <range>` and `git diff <range> --stat`
4. Get the diff: `git diff <range>`
5. Get changed files: `git diff <range> --name-only`
6. Add range metadata to the context JSON under `commit_range` (see Context JSON Format)
7. Run the Shared Review Pipeline with `--type code`

### Plan Review Workflow (`plan <path>` or `<file>.md` or detected plan)

1. Find the plan file:
   - If explicit path provided → Use that path
   - If `<file>.md` provided → Use that file
   - If detected via Smart Detection → Use detected path
   - If none → Check most recent in `~/.claude/plans/`
2. **Read the FULL plan content** and include it in the context JSON under the `plan_content` field
3. Parse plan for file paths and read those files into `file_contents`
4. Run the Shared Review Pipeline with `--type plan`

**IMPORTANT**: Do NOT pass `--plan-file` as a separate argument. The plan content MUST be included directly in the context JSON file under the `plan_content` key.

### PR Review Workflow (`pr <number>`)

1. Extract PR number from arguments
2. Fetch PR info: `gh pr view <number> --json title,body,author,baseRefName,headRefName,files,additions,deletions`
3. Get PR diff: `gh pr diff <number>`
4. Add the PR info to the context JSON under `pr_metadata` (see Context JSON Format)
5. Run the Shared Review Pipeline with `--type pr`

---

## Include Conversation Context

Review the conversation history and include relevant context that explains the developer's intent:

1. **Original Request** - What did the user ask you to do? (1-2 sentences)
2. **Approach Notes** - Key decisions, constraints, or tradeoffs mentioned (bullet points)
3. **Relevant Exchanges** - 3-5 most relevant user messages and your responses that explain the changes:
   - Why this approach was chosen
   - Constraints or requirements mentioned
   - Errors encountered and how they were addressed
4. **Previous Review Findings** - If `/h3` was run earlier in this session, summarize key findings

**Selection criteria for relevant exchanges:**
- Messages that explain WHY changes were made
- Messages discussing tradeoffs or alternatives
- Messages mentioning constraints, requirements, or edge cases
- Messages about errors or bugs being fixed
- Skip: casual messages, unrelated topics, raw tool outputs

**Limits:**
- Maximum 3-5 exchanges (user message + your response = 1 exchange)
- Keep each message under 500 characters (truncate if needed)
- Total conversation context should not exceed ~2K tokens

---

## Find Cross-File Dependencies

For code and PR reviews (not plan reviews), search for files that import or reference the changed files. This helps reviewers catch breaking changes. Cap at 20 dependent files, capture the import line(s) plus ~3 lines of context each, store them in the context JSON as `dependent_files`, and omit the key entirely when none are found.

The exact grep command, filtering rules, and the over-20 prioritization scheme live in `${CLAUDE_SKILL_DIR}/references/cross-file-dependencies.md`. Read that file when you reach step 4 of the Shared Review Pipeline.

---

## Context JSON Format

**IMPORTANT**: All content must be included IN the context JSON file. Do NOT pass separate file arguments.

The full schema (all top-level keys, per-review-type requirements, example values) is in `${CLAUDE_SKILL_DIR}/references/context-json-format.md`. Read that file when you assemble the context JSON (it is not needed before then). Top-level keys: `review_type`, `conversation_context`, `plan_content` (plan reviews), `diff` + `changed_files` + `file_contents` (code/pr reviews), `documentation`, `test_files`, `dependent_files`, `pr_metadata`, `commit_metadata`, `commit_range`.

---

## Temp File Handling

**CRITICAL: Use Bash to write the context JSON — NEVER use the Write tool.**

**CRITICAL: Use a UNIQUE filename per review session to prevent concurrent reviews from overwriting each other.**

Generate a unique context file path at the start of each review using a timestamp and random suffix, then use `$H3_CONTEXT_FILE` for all subsequent commands in that review session:

```bash
H3_CONTEXT_FILE="/tmp/h3-context-$(date +%s)-$RANDOM.json"
cat > "$H3_CONTEXT_FILE" << 'CONTEXT_EOF'
{...the JSON...}
CONTEXT_EOF
```

This ensures the **only** user-facing prompt is the cost estimate confirmation. Do NOT use the Write tool for this file.

---

## Process and Act on the Review

### Step 1: Display the Review

```markdown
## Heavy3 Code Audit [Single/Council] (from [model name(s)])

[Output from the review script]
```

If council mode, show all 3 reviews clearly labeled with their roles.

### Step 2: Synthesize (Council Mode Only) - COMPARISON TABLE REQUIRED

For council reviews, YOU (Claude) MUST synthesize the 3 reviews. Do NOT just list them sequentially, and do NOT skip the comparison table even if the reviews are similar.

**CRITICAL REQUIREMENT: The 3-column comparison table is Heavy3's TRADEMARK FEATURE.** The synthesis must contain: the 3-column comparison table, consensus issues (flagged by 2+ reviewers), notable findings per reviewer, a final recommendation (APPROVE / APPROVE WITH CHANGES / REQUEST CHANGES), and a priority action list.

The full synthesis template, checklist, and rationale live in `${CLAUDE_SKILL_DIR}/references/council-synthesis.md`. Read that file when a `--council` review completes, before writing the synthesis.

### Step 3: Analyze and Assess Each Finding

```markdown
## My Assessment

| # | Issue | Reviewer Says | My Take | Action |
|---|-------|---------------|---------|--------|
| 1 | [Brief] | [Concern] | AGREE/PARTIAL/DISAGREE | [What to do] |
```

### Step 4: Propose Actionable Items

```markdown
## Proposed Actions

**Immediate fixes I can make:**
1. [Fix with file:line]

**Needs your decision:**
1. [Tradeoff to discuss]

**No action needed:**
1. [Why disagree]
```

### Step 5: Ask User for Approval

```markdown
**What would you like me to do?**

1. **Fix all** - Apply all immediate fixes
2. **Fix specific items** - Tell me which (e.g., "fix 1, 3")
3. **Discuss first** - Talk through items
4. **Skip** - No changes
```

### Step 6: Call for Contributions

After presenting results and user makes a choice (or after any review completes), display:

```markdown
---

**Like Heavy3 Code Audit?** [Star on GitHub](https://github.com/heavy3-ai/code-audit) | [Contribute](https://github.com/heavy3-ai/code-audit/issues) | Share with your team
```

**When to show:**
- After user selects an action (fix all, fix specific, discuss, skip)
- After a single review completes (non-council mode)
- At the end of module-by-module reviews

**Keep it brief** - one line, non-intrusive.

---

## Important Guidelines

- **Single confirmation only**: Gather all context and save the temp JSON file BEFORE showing the cost estimate. The cost estimate is the ONLY prompt requiring user input. Use Bash to write temp files to `/tmp/` with a unique name per session (see Temp File Handling) — the Write tool is not available.
- **Be honest**: Disagree with reviewers when warranted
- **Be specific**: Exact files and line numbers
- **Don't auto-fix**: ALWAYS wait for user approval
- **Prioritize**: Security/bugs first, style last
- **PR reviews**: Highlight blocking issues if REQUEST CHANGES
- **Comparison table**: ALWAYS show the 3-column table for council reviews
