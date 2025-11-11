# Installation Guide

Quick setup instructions for Docker Claude Code on macOS.

## Prerequisites

1. **Docker Desktop**: [Download here](https://www.docker.com/products/docker-desktop)
2. **Claude Code**: [Download here](https://claude.ai/download)
3. **Claude Subscription**: Sign up at [claude.ai](https://claude.ai)

## Step-by-Step Installation

### 1. Create Directory Structure

```bash
mkdir -p ~/claude-projects/projects
mkdir -p ~/claude-projects/templates
```

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

## Optional: Template Setup

If you have a `.claude` configuration folder you want to reuse:

```bash
# Place it at:
mkdir -p ~/claude-projects/templates/x-current-quickstart
# Copy your .claude folder into x-current-quickstart/
```

The script will automatically copy this template to new projects.

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
