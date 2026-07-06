# Council Synthesis Template and Checklist

Read this file when a `--council` review completes, before writing Claude's Synthesis (Step 2 of Process and Act on the Review in SKILL.md). Not needed for single-model reviews.

## Synthesis Template

```markdown
## Claude's Synthesis

### Comparison of All Three Reviews

| Aspect | Correctness (GPT 5.5) | Performance (Gemini 3.1) | Security (Grok 4) |
|--------|----------------------|----------------------|---------------------|
| **Focus** | Bugs, Logic, Edge Cases | Scaling, Memory, N+1 | Vulnerabilities, Auth |
| **Findings** | FAIL: 1 bug: null check missing | WARN: Potential N+1 query | OK: No XSS, SQL injection |
| **Verdict** | REQUEST CHANGES | APPROVE WITH NOTES | APPROVE |

Legend: OK = No issues | WARN = Warning/Concern | FAIL = Critical issue

### Consensus Issues (Flagged by 2+ reviewers)
- [Issue that multiple reviewers agree on]

### Notable Findings (From individual reviewers)
- **Correctness Expert**: [Specific finding]
- **Security Analyst**: [Specific finding]
- **Performance Critic**: [Specific finding]

### Final Recommendation
[Your overall assessment: APPROVE / APPROVE WITH CHANGES / REQUEST CHANGES]

**Priority Actions:**
1. [Most important fix]
2. [Second priority]
3. [Lower priority]
```

## Checklist for Council Synthesis

- [ ] 3-column comparison table with all aspects
- [ ] Legend explaining the OK / WARN / FAIL markers
- [ ] Consensus issues (flagged by 2+ reviewers)
- [ ] Notable findings from each reviewer
- [ ] Final recommendation (APPROVE / APPROVE WITH CHANGES / REQUEST CHANGES)
- [ ] Priority action list

**DO NOT** just list the three reviews sequentially without synthesis.
**DO NOT** skip the comparison table even if reviews are similar.
**DO** actively identify where reviewers agree or disagree.

## Why This Matters: Synthesis Creates Action

The comparison table is just the start. What makes Heavy3 valuable is **turning diverse perspectives into actionable next steps**:

1. **Consensus = High Confidence**: When 2+ reviewers flag the same issue, prioritize it
2. **Unique Insights = Coverage**: Each specialist catches things others miss
3. **Disagreement = Discussion Point**: When reviewers conflict, surface it for human judgment
4. **Priority Actions = Clear Path Forward**: Don't just report, recommend what to fix first

The goal isn't just to show three opinions. It's to synthesize them into **one clear action plan** the developer can execute immediately.
