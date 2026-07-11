# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Heavy3 Code Audit (`/h3`) is a Claude Code skill providing AI-powered multi-model code reviews via OpenRouter. Modes:
- **Single (default)**: DeepSeek V4 Pro, up to 200K tokens
- **Council**: 3-model parallel review (GPT 5.6 Sol + Gemini 3.1 Pro + Grok 4.5), up to 200K tokens
- **Free**: Rotating free models from OpenRouter

## Local Development Setup

```bash
# Symlink for Claude Code discovery
mkdir -p ~/.claude/skills && ln -sf "$(pwd)/skill" ~/.claude/skills/h3

# Make scripts executable
chmod +x skill/scripts/*.py

# Install dependency
pip install requests

# Required env var
export OPENROUTER_API_KEY="your-key"
```

## Architecture

The skill uses YAML frontmatter + markdown format in `SKILL.md` to define the skill manifest that Claude reads during `/h3` execution.

```
skill/
├── SKILL.md              # Skill manifest: YAML frontmatter + instructions for Claude
├── config.json           # User config (tier, model, license_key, context limits)
└── scripts/
    ├── review.py         # Lite mode: OpenRouter API calls, prompt templates
    ├── council.py        # Pro mode: 3-parallel reviews with local license check
    ├── license.py        # License activation and verification
    └── list-free-models.py # OpenRouter free model discovery
```

**Data Flow:**
1. User invokes `/h3 code|plan|pr <number>` in Claude Code
2. Claude reads SKILL.md, gathers context (git diff, files, docs)
3. Context compiled to JSON and passed to review script via `--context-file`
4. Script calls OpenRouter API, returns review
5. Claude synthesizes findings, proposes fixes for user approval

## Testing the Skill

### Automated Tests (pytest)

```bash
# Install test dependencies
cd skill && pip install -r requirements-dev.txt

# Run all tests
cd scripts && python -m pytest tests/ -v

# Run specific test file
python -m pytest tests/test_context.py -v
python -m pytest tests/test_review.py -v
python -m pytest tests/test_streaming.py -v
python -m pytest tests/test_council.py -v
```

### Manual Integration Tests

After symlinking, test in Claude Code:
- `/h3 code` - Review uncommitted changes
- `/h3 plan` - Review implementation plan
- `/h3 pr <number>` - Review GitHub PR
- `/h3 --council` - 3-model council review

### Testing Requirements

**Every new feature MUST have:**
1. **Unit tests** - Test individual functions/modules in isolation
2. **Integration tests** (for major features) - Test the full workflow end-to-end

**Before merging:**
- All existing tests must pass: `python -m pytest tests/ -v`
- New tests must be added for new functionality
- Test coverage should not decrease

## Key Implementation Details

- **Model shortcuts**: `--model gpt`, `--model deepseek`, `--model free` resolve to full OpenRouter model IDs
- **Context limits**: Single mode 200K tokens (~800K chars); Council mode 200K tokens per reviewer (per-model overrides in `max_context_by_model`)
- **Large changes**: SKILL.md instructs Claude to detect >50 files or >10K lines and break into module-by-module reviews
- **PR review**: Uses `gh pr view` and `gh pr diff` for GitHub integration
- **Council synthesis**: Council mode outputs a comparison table across 3 reviewer perspectives (Correctness/Security/Performance)
