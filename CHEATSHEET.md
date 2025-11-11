# Quick Reference

Common commands and workflows for Docker Claude Code.

## Essential Commands

### Project Management

```bash
# Create new project
new-code-proj my-project

# Navigate to project
cd ~/claude-projects/projects/my-project

# Start container (with alias)
claude-start

# Start container (without alias)
docker-compose up -d
docker exec -it my-project bash

# Stop container (with alias)
claude-stop

# Stop container (without alias)
docker-compose down
```

### Inside Container

```bash
# Run Claude Code
claude --dangerously-skip-permissions

# Or without flag
claude

# Check Claude version
claude --version

# Exit container
exit
# or press Ctrl+D
```

### Docker Management

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Stop all containers
docker stop $(docker ps -q)

# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Check disk usage
docker system df

# Full cleanup (be careful!)
docker system prune -a
```

## Daily Workflow

### Morning
```bash
cd ~/claude-projects/projects/current-project
claude-start
claude --dangerously-skip-permissions
```

### During Day
- Edit files in `workspace/` on your Mac
- Changes appear instantly in container
- Close terminal without stopping container if needed

### Evening
```bash
exit              # Exit container
claude-stop       # Stop container to free RAM
```

## File Operations

### Working with Files

```bash
# Files go here on your Mac:
~/claude-projects/projects/my-project/workspace/

# They appear here in container:
/workspace/
```

### View Hidden Files in Container

```bash
ls -la /workspace
```

### View Hidden Files on Mac (Finder)

Press `Cmd + Shift + .` (period)

## Troubleshooting

### Quick Fixes

```bash
# Rebuild container after Dockerfile changes
docker-compose build --no-cache
claude-start

# Reset project (keeps workspace files)
claude-stop
docker-compose build --no-cache
claude-start

# Check container logs
docker logs my-project-name

# Check if Claude installed
docker exec -it my-project-name which claude
```

### Common Issues

**"command not found: new-code-proj"**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**"claude: command not found" in container**
```bash
export PATH="$HOME/.local/bin:$PATH"
claude --dangerously-skip-permissions
```

**"Cannot connect to Docker daemon"**
- Open Docker Desktop
- Wait for it to start (green icon in menu bar)

**Files not syncing**
- Check you're putting files in `workspace/` folder
- Restart Docker Desktop if needed

## Aliases Reference

Add to `~/.zshrc`:

```bash
# Start container and enter
alias claude-start='docker-compose up -d && docker exec -it $(basename "$PWD") bash'

# Stop container
alias claude-stop='docker-compose down'

# Quick navigate to projects
alias cdp='cd ~/claude-projects/projects'

# List all Claude containers
alias claude-ps='docker ps -a | grep claude'

# Stop all Claude containers
alias claude-stop-all='docker ps -a | grep claude | awk "{print \$1}" | xargs docker stop'
```

## Customization

### Change Project Directory

Edit `~/bin/new-code-proj`:
```bash
PROJECT_PATH="$HOME/your/custom/path/$PROJECT_NAME"
```

### Change Template Location

Edit `~/bin/new-code-proj`:
```bash
TEMPLATE_CLAUDE="$HOME/your/template/path/.claude"
```

### Add Tools to Container

Edit Dockerfile in `~/bin/new-code-proj`:
```dockerfile
RUN apt-get update && apt-get install -y \
    curl ca-certificates git wget bash sudo \
    nodejs npm \           # Add Node.js
    python3 python3-pip \  # Add Python
    && rm -rf /var/lib/apt/lists/*
```

## Resource Management

### Check What's Using Space

```bash
# Docker disk usage
docker system df

# Detailed breakdown
docker system df -v

# List large images
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h
```

### Clean Up Space

```bash
# Remove unused containers
docker container prune

# Remove unused images  
docker image prune -a

# Remove everything (careful!)
docker system prune -a --volumes

# Before cleanup:
docker system df

# After cleanup:
docker system df
```

## Multiple Projects

### Run Multiple Projects Simultaneously

```bash
# Terminal 1
cd ~/claude-projects/projects/project-a
claude-start

# Terminal 2  
cd ~/claude-projects/projects/project-b
claude-start

# Terminal 3
cd ~/claude-projects/projects/project-c
claude-start
```

Each runs independently with its own resources.

### Switch Between Projects

```bash
# Stop current
claude-stop

# Switch directory
cd ~/claude-projects/projects/other-project

# Start new
claude-start
```

## Backup & Restore

### Backup Project

```bash
# Backup workspace only (recommended)
cd ~/claude-projects/projects
tar -czf my-project-backup.tar.gz my-project/workspace/

# Backup entire project
tar -czf my-project-full.tar.gz my-project/
```

### Restore Project

```bash
# Extract backup
cd ~/claude-projects/projects
tar -xzf my-project-backup.tar.gz

# Start container
cd my-project
claude-start
```

## Advanced

### Run Commands Without Entering Container

```bash
# Run single command
docker exec -it my-project bash -c "ls -la /workspace"

# Run Claude command
docker exec -it my-project bash -c "cd /workspace && claude --help"
```

### Mount Additional Volumes

Edit `docker-compose.yml`:
```yaml
volumes:
  - ./workspace:/workspace
  - ~/.anthropic:/home/developer/.anthropic
  - ~/my-other-folder:/mnt/other  # Add this
```

### Set Environment Variables

Add to `.env` file:
```bash
PROJECT_NAME=my-project
MY_CUSTOM_VAR=value
```

Or in `docker-compose.yml`:
```yaml
environment:
  - MY_VAR=value
  - ANOTHER_VAR=123
```

## Getting Help

- [Main README](README.md) - Full documentation
- [Installation Guide](INSTALL.md) - Setup instructions
- [GitHub Issues](https://github.com/chrisabra-co/docker-claude-mac/issues) - Report problems
- [Docker Documentation](https://docs.docker.com/) - Docker help
- [Claude Documentation](https://docs.claude.ai/) - Claude help
