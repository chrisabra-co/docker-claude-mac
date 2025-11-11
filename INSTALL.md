# Installation Guide

Quick setup instructions for Docker Claude Code on macOS.

## Prerequisites

1. **Docker Desktop**: [Download here](https://www.docker.com/products/docker-desktop)
2. **Claude Code**: [Download here](https://claude.ai/download)
3. **Claude Subscription**: Sign up at [claude.ai](https://claude.ai)

## Step-by-Step Installation

### 1. Create Directory Structure

**Default setup (recommended):**

```bash
mkdir -p ~/claude-projects/projects
mkdir -p ~/claude-projects/templates
```

**Using existing project folders:**

If you prefer to use your existing project directory:

```bash
# Example: Use ~/projects instead
mkdir -p ~/projects
mkdir -p ~/claude-projects/templates
```

> **Note:** If using a custom directory, you'll need to edit `~/bin/new-code-proj` after step 2 and change the `PROJECT_PATH` variable to point to your location.

**Setting up templates (optional):**

The template folder contains a `.claude` configuration that gets **copied once** to new projects at creation time.

> **⚠️ Security:** Never include API keys, passwords, tokens, or secrets in templates. They'll be copied to every project.

> **📋 Important:** Templates are copied at project creation, not synced. Updating the template only affects future projects.

```bash
# Create template structure
mkdir -p ~/claude-projects/templates/proj-quickstart/.claude

# Copy from existing project (if you have one)
cp -r ~/path/to/existing/.claude/* ~/claude-projects/templates/proj-quickstart/.claude/

# Or create subdirectories for commands, agents, etc.
mkdir -p ~/claude-projects/templates/proj-quickstart/.claude/commands
mkdir -p ~/claude-projects/templates/proj-quickstart/.claude/agents
```

**Example: Create a review command**
```bash
cat > ~/claude-projects/templates/proj-quickstart/.claude/commands/review.md << 'EOF'
Review this code for:
- Security vulnerabilities (XSS, SQL injection, etc.)
- Performance issues and bottlenecks
- Best practices and code quality
- Potential bugs and edge cases

Provide specific suggestions for improvement.
EOF
```

Template can include:
- Custom slash commands (`.claude/commands/`)
- Specialized agents (`.claude/agents/`)
- MCP server configs (`.claude/mcp.json`)
- Project settings (`.claude/settings.json`)
- Reusable prompts (`.claude/prompts/`)

**Starting without a template is fine!** New projects will have an empty workspace ready to build from scratch.

### 2. Install the Script

```bash
# Create bin directory
mkdir -p ~/bin

# Download and install the script
curl -o ~/bin/new-code-proj https://raw.githubusercontent.com/chrisabra-co/docker-claude-mac/main/new-code-proj

# Make it executable
chmod +x ~/bin/new-code-proj
```

### 3. Add to PATH

Add this line to your `~/.zshrc` (or `~/.bash_profile` for bash):

```bash
export PATH="$HOME/bin:$PATH"
```

Then reload:

```bash
source ~/.zshrc
```

### 4. Set Up Aliases (Optional)

Add these to your `~/.zshrc`:

```bash
# Quick start Claude Code in current directory's container
alias claude-start='docker-compose up -d && docker exec -it $(basename "$PWD") bash'

# Quick stop
alias claude-stop='docker-compose down'
```

Reload:

```bash
source ~/.zshrc
```

### 5. Authenticate Claude

```bash
claude login
```

This stores your credentials in `~/.anthropic/` which gets mounted into containers.

## Verify Installation

```bash
# Check if script is available
which new-code-proj

# Should output: /Users/YOUR_USERNAME/bin/new-code-proj
```

## Create Your First Project

```bash
new-code-proj test-project
cd ~/claude-projects/projects/test-project
claude-start
claude --dangerously-skip-permissions
```

## Template Usage

If you set up a template in step 1, it will automatically be copied to every new project you create with `new-code-proj`.

**Verify your template:**
```bash
ls -la ~/claude-projects/templates/proj-quickstart/.claude/
```

**Updating templates:**
- Modify files in `~/claude-projects/templates/proj-quickstart/.claude/`
- Changes affect **new projects only** (not existing ones)
- To update an existing project, copy manually:
  ```bash
  cp -r ~/claude-projects/templates/proj-quickstart/.claude ~/claude-projects/projects/my-project/workspace/
  ```

**Template location in Docker:**
```
Your Mac                              Docker Container
~/claude-projects/projects/           (inside container)
  my-project/
    workspace/              ←→         /workspace/
      .claude/                         .claude/    ← Claude Code sees this
```

## Troubleshooting

### "command not found: new-code-proj"

Check your PATH:
```bash
echo $PATH | grep bin
```

If ~/bin isn't there, add to ~/.zshrc:
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### "Docker daemon not running"

Open Docker Desktop and ensure it's running (look for Docker icon in menu bar).

### "claude: command not found" (on Mac)

Install Claude Code from [https://claude.ai/download](https://claude.ai/download)

Then authenticate:
```bash
claude login
```

## Next Steps

- Read the [main README](README.md) for detailed usage
- Check [troubleshooting section](README.md#troubleshooting) if you hit issues
- Customize the script for your preferred directory structure

## Need Help?

Open an issue at [https://github.com/chrisabra-co/docker-claude-mac/issues](https://github.com/chrisabra-co/docker-claude-mac/issues)
