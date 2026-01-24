# Installation Guide - Windows

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

```powershell
git clone https://github.com/heavy3-ai/code-audit.git
cd code-audit
```

### 3. Symlink Skill to Claude Code (Recommended)

Using a symlink means updates via `git pull` are instantly available.

**PowerShell (Run as Administrator):**
```powershell
# Create skills directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"

# Create symlink (requires admin privileges)
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\h3" -Target "$(Get-Location)\skill" -Force

# Verify the symlink
Get-Item "$env:USERPROFILE\.claude\skills\h3"
```

**Alternative: Copy (if you can't run as admin)**

If you can't create symlinks, use copy instead (you'll need to re-copy after `git pull`):

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse -Force skill "$env:USERPROFILE\.claude\skills\h3"
```

### 4. Install Python Dependency

```powershell
pip install requests
```

### 5. Set Environment Variable

**PowerShell (Permanent - Recommended):**
```powershell
[System.Environment]::SetEnvironmentVariable("OPENROUTER_API_KEY", "your-key-here", "User")
```

**Command Prompt (Permanent):**
```cmd
setx OPENROUTER_API_KEY "your-key-here"
```

**Important:** Restart your terminal after setting the environment variable.

### 6. Verify Installation

```powershell
# Check the skill exists (symlink or folder)
Get-Item "$env:USERPROFILE\.claude\skills\h3"

# Test the Python script
python "$env:USERPROFILE\.claude\skills\h3\scripts\review.py" --help
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

**If using symlink:** Just pull the latest changes:
```powershell
cd code-audit
git pull
```

**If using copy:** Pull and re-copy:
```powershell
cd code-audit
git pull
Copy-Item -Recurse -Force skill "$env:USERPROFILE\.claude\skills\h3"
```

## Uninstalling

```powershell
# Remove the symlink or folder
Remove-Item -Force "$env:USERPROFILE\.claude\skills\h3"

# If it's a directory (not symlink), use -Recurse
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\h3"
```

## Troubleshooting

### Python not found

Make sure Python is in your PATH. You can verify with:
```powershell
python --version
```

If not found, reinstall Python and check "Add Python to PATH" during installation.

### Symlink requires admin

Windows requires administrator privileges to create symbolic links. Either:
1. Run PowerShell as Administrator
2. Use the copy method instead

### Environment variable not working

1. Restart your terminal completely (close all windows)
2. Verify the variable is set:
   ```powershell
   echo $env:OPENROUTER_API_KEY
   ```
3. If still not working, try setting it via System Properties > Environment Variables
