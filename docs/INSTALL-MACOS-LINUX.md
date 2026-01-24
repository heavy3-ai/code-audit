# Installation Guide - macOS / Linux

## Prerequisites

- Claude Code CLI installed
- Python 3.8+
- Git
- OpenRouter API key

## Step-by-Step Installation

### 1. Get an OpenRouter API Key

1. Go to https://openrouter.ai/keys
2. Create an account or sign in
3. Create a new API key
4. Copy the key (starts with `sk-or-...`)

### 2. Clone the Repository

```bash
git clone https://github.com/heavy3-ai/code-audit.git
cd code-audit
```

### 3. Symlink Skill to Claude Code

Using a symlink means updates via `git pull` are instantly available.

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Create symlink (use absolute path)
ln -sf "$(pwd)/skill" ~/.claude/skills/h3

# Make scripts executable
chmod +x skill/scripts/*.py

# Verify the symlink
ls -la ~/.claude/skills/h3
```

### 4. Install Python Dependency

```bash
pip install requests

# Or with pip3 if you have multiple Python versions
pip3 install requests
```

### 5. Set Environment Variable

**For Zsh (default on macOS):**
```bash
echo 'export OPENROUTER_API_KEY="your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

**For Bash:**
```bash
echo 'export OPENROUTER_API_KEY="your-key-here"' >> ~/.bashrc
source ~/.bashrc
```

### 6. Verify Installation

```bash
# Check the symlink points to the right place
ls -la ~/.claude/skills/h3

# Should show something like:
# h3 -> /path/to/code-audit/skill

# Test the Python script
python3 ~/.claude/skills/h3/scripts/review.py --help
```

## Usage

Open Claude Code and type:

```bash
# Smart detection - figures out what to review
/h3                      # Auto-detect: changes, plan, or asks
/h3 --free               # Same, but use free model
/h3 --council            # Same, but use Pro 3-model council

# Explicit targets
/h3 pr 123               # Review PR #123
/h3 plan.md              # Review a specific plan
/h3 HEAD~3..HEAD         # Review last 3 commits

# Scope modifiers
/h3 --staged             # Force review of only staged changes
/h3 --commit             # Force review of the last commit
```

## Updating

Since you're using a symlink, just pull the latest changes:

```bash
cd /path/to/code-audit
git pull
```

That's it! The symlink automatically reflects the updated files.

## Uninstalling

```bash
# Remove the symlink (doesn't delete the source files)
rm ~/.claude/skills/h3

# Optionally, delete the cloned repository
rm -rf /path/to/code-audit
```
