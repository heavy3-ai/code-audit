# Heavy3 Code Audit (`/h3`)

**Sponsored by [Heavy3.ai](https://heavy3.ai) - Multi-model AI for high-stakes decisions**

Agent skill that uses multi-model consensus to review plans, code, and PRs for coding agents (Claude Code, Cursor, Codex CLI, Gemini CLI, Antigravity).

AI-powered code reviews via OpenRouter. Works with any CLI that supports the [agent skill standard](https://code.claude.com/docs/en/skills).

**100% Free and Open Source** - All features unlocked, no license required.

## Features

- **Smart Detection** - Just run `/h3` and it figures out what to review
- **Plan Review** - Automatically detects if you just finished planning
- **Code Review** - Reviews uncommitted changes or commit ranges
- **PR Review** - Review GitHub pull requests with `/h3 pr 123`
- **Multi-Commit Review** - Review features spanning multiple commits (`/h3 HEAD~3..HEAD`)
- **Context-aware** - Includes docs, tests, and file contents automatically
- **Conversation-aware** - Understands your intent from conversation history

## Modes

| Mode | Model | Est. Cost | Command |
|------|-------|-----------|---------|
| **Single** (default) | Kimi K2.5 | ~$0.01/review | `/h3` |
| **Free** | Rotating | $0.00 | `/h3 --free` |
| **Council** | GPT 5.2 + Gemini 3 Pro + Grok 4 | ~$0.10-0.20 | `/h3 --council` |

Council uses 3 specialized models with web search, then synthesizes findings.

## Quick Start

```bash
# Clone and symlink
git clone https://github.com/heavy3-ai/code-audit.git
cd code-audit
mkdir -p ~/.claude/skills
ln -sf "$(pwd)/skill" ~/.claude/skills/h3
chmod +x skill/scripts/*.py

# Install dependency
pip install requests

# Set API key (choose one option):
# Option 1: Create .env file in skill directory
echo 'OPENROUTER_API_KEY=your-key-here' > ~/.claude/skills/h3/.env

# Option 2: Set environment variable (add to ~/.zshrc for persistence)
export OPENROUTER_API_KEY="your-key-here"
```

See [docs/INSTALL-MACOS-LINUX.md](docs/INSTALL-MACOS-LINUX.md) or [docs/INSTALL-WINDOWS.md](docs/INSTALL-WINDOWS.md) for full instructions.

## Usage

```bash
# Smart detection (recommended) - figures out what to review
/h3                      # Auto-detect: changes, plan, or ask
/h3 --free               # Same, but use free model
/h3 --council            # Same, but use Pro 3-model council

# Explicit targets (when you know what you want)
/h3 pr 123               # Review PR #123
/h3 plan.md              # Review a specific plan file
/h3 HEAD~3..HEAD         # Review last 3 commits

# Scope modifiers
/h3 --staged             # Review only staged changes
/h3 --commit             # Review only the last commit
```

### Smart Detection Flow

1. **Has uncommitted changes?** → Confirms, then reviews changes
2. **No changes + plan detected?** → Confirms, then reviews plan
3. **No changes + no plan?** → Asks what to review (latest commit, range, etc.)

## Council Mode

3-model council with specialized reviewers (all free):
- **GPT 5.2** - Correctness (bugs, logic errors)
- **Gemini 3 Pro** - Performance (N+1 queries, memory leaks)
- **Grok 4** - Security (vulnerabilities, auth)

Just run `/h3 --council` to use.

## Requirements

- Python 3.8+
- OpenRouter API key ([get one here](https://openrouter.ai/keys))

## Documentation

- [Installation (macOS/Linux)](docs/INSTALL-MACOS-LINUX.md)
- [Installation (Windows)](docs/INSTALL-WINDOWS.md)
- [Configuration Reference](docs/CONFIGURATION.md)
- [Methodology (White Paper)](docs/METHODOLOGY.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Contributing](CONTRIBUTING.md)

## License

MIT License - see [LICENSE](LICENSE)
