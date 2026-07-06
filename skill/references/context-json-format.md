# Context JSON Format

Read this file when you assemble the context JSON for a review (Shared Review Pipeline step 6 in SKILL.md). It is not needed before assembly.

**IMPORTANT**: All content must be included IN the context JSON file. Do NOT pass separate file arguments to the review scripts (no `--plan-file`; only `--type` and `--context-file`).

## Full Schema

```json
{
  "review_type": "plan" | "code" | "pr",
  "conversation_context": {
    "original_request": "Brief summary of what user originally asked for",
    "approach_notes": "Key decisions made during implementation",
    "relevant_exchanges": [
      {"role": "user", "content": "Can you add validation to the form?"},
      {"role": "assistant", "content": "I'll add Zod validation. Using inline validation rather than form-level because..."}
    ],
    "previous_review_findings": "Summary of any prior /h3 review in this session"
  },
  "plan_content": "# Full plan markdown content (REQUIRED for plan reviews)",
  "diff": "git diff output (for code/pr reviews)",
  "changed_files": ["path1", "path2"],
  "file_contents": {
    "path1": "full file content...",
    "path2": "full file content..."
  },
  "documentation": {
    "CLAUDE.md": "...",
    "documents/feature.md": "..."
  },
  "test_files": {
    "path1.test.ts": "..."
  },
  "dependent_files": {
    "src/components/UserList.tsx": "import { validateEmail } from '../utils';\n...\nconst isValid = validateEmail(user.email);",
    "src/api/handlers.ts": "import { calculateTotal } from '../utils';\n...\nreturn calculateTotal(cart.items);"
  },
  "pr_metadata": {
    "number": 123,
    "title": "...",
    "body": "...",
    "author": "...",
    "base_branch": "main",
    "head_branch": "feature",
    "additions": 100,
    "deletions": 50
  },
  "commit_metadata": {
    "hash": "abc123...",
    "subject": "feat: Add user authentication",
    "author": "John Doe",
    "date": "2025-01-25"
  },
  "commit_range": {
    "range": "HEAD~3..HEAD",
    "commits": [
      {"hash": "abc123", "subject": "feat: Add login", "date": "2025-01-28"},
      {"hash": "def456", "subject": "fix: Edge case", "date": "2025-01-29"}
    ],
    "total_commits": 2
  }
}
```

## Per-Review-Type Requirements

| Key | plan | code / --commit / range | pr |
|-----|------|--------------------------|-----|
| `review_type` | required | required (`"code"`) | required |
| `plan_content` | REQUIRED | omit | omit |
| `diff`, `changed_files`, `file_contents` | file_contents only (files the plan touches) | required | required |
| `documentation`, `conversation_context` | include | include | include |
| `test_files` | omit | include | include |
| `dependent_files` | omit | include; omit the key entirely when none found | include; omit when none found |
| `pr_metadata` | omit | omit | required |
| `commit_metadata` | omit | `--commit` reviews only | omit |
| `commit_range` | omit | range reviews only | omit |
