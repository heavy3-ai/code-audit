# Large Changes: Module Selection and Module-by-Module Review

Read this file only when the Step 0 size check in SKILL.md trips: more than 50 changed files, more than 10,000 additions + deletions, or total characters over the active model's budget (see Context Limits in SKILL.md).

## Step 1: Stop and Present Module Options

If estimated context exceeds the model's budget, **DO NOT proceed automatically**. Present module options to user:

```markdown
## Large Change Detected - Module Selection Required

| Metric | Value | Limit |
|--------|-------|-------|
| Changed files | [X] | ~50 |
| Lines changed | +[X]/-[Y] | ~10,000 |
| Est. chars | ~[X]K | [model budget from config.json] |

I found [X] changed files across these areas:

| # | Module | Files | Est. Tokens | Description |
|---|--------|-------|-------------|-------------|
| 1 | src/components | 18 | ~25K | UI components |
| 2 | src/utils | 12 | ~15K | Utility functions |
| 3 | src/api | 10 | ~20K | API handlers |
| 4 | tests | 5 | ~8K | Test files |

**How would you like to proceed?**
1. Review modules separately (4 reviews, ~$X.XX total)
2. Combine modules 2+3 into one review (3 reviews)
3. Review all together (will truncate to fit limit)
4. Custom grouping (tell me which modules to combine)
```

## Step 2: Module-by-Module Review Workflow

After user selects grouping:

1. **Create progress tracking table**:
```markdown
## Review Progress

| Module | Files | Status | Key Findings |
|--------|-------|--------|--------------|
| src/components | 18 | IN PROGRESS | - |
| src/utils + src/api | 22 | PENDING | - |
| tests | 5 | PENDING | - |
```

2. **Review each module group**:
   - Run the review for current module
   - Update the progress table with status and key findings
   - After each review, ask: "Continue to next module? (y/n)"
   - If user says no, offer to save progress and resume later

3. **Update progress after each module**:
```markdown
## Review Progress (Updated)

| Module | Files | Status | Key Findings |
|--------|-------|--------|--------------|
| src/components | 18 | COMPLETE | 2 security, 1 perf issue |
| src/utils + src/api | 22 | IN PROGRESS | - |
| tests | 5 | PENDING | - |
```

4. **Final cross-module synthesis** (after all modules reviewed):
```markdown
## Cross-Module Summary

### All Issues by Category

**Security Issues (across all modules):**
- [src/components:42] XSS vulnerability in user input
- [src/api:15] Missing authentication check

**Performance Issues (across all modules):**
- [src/components:88] N+1 query in list render

**Correctness Issues (across all modules):**
- [src/utils:23] Off-by-one error in pagination

### Recommended Fix Priority
1. **CRITICAL**: [Security issue from module 1]
2. **HIGH**: [Performance issue from module 2]
3. **MEDIUM**: [Other issues...]

### Cross-Module Concerns
- [Any issues that span multiple modules]
- [Architectural concerns from combined view]
```

Remember the contribution footer (SKILL.md, Step 6) at the end of module-by-module reviews.
