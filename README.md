# Docker Claude Code Setup for Mac

A streamlined Docker setup for running Claude Code with `--dangerously-skip-permissions` on macOS. This setup provides isolated project environments with live file syncing between your Mac and Docker containers.

## Why This Setup?

- **Isolated Environments**: Each project runs in its own Docker container
- **Skip Permissions**: Run Claude Code with `--dangerously-skip-permissions` flag (not possible as root)
- **Live File Sync**: Edit files on your Mac, changes appear instantly in the container
- **Template Support**: Auto-copy `.claude` configuration folders to new projects
- **One-Command Setup**: Create new projects with a single command

## Prerequisites

- macOS (M1/M2/Intel)
- [Docker Desktop](https://www.docker.com/products/docker-desktop) installed and running
- [Claude Code](https://claude.ai/download) installed on your Mac
- Claude Code subscription (authenticated via `claude login` on your Mac)

## Installation

### 1. Clone This Repository

```bash
git clone https://github.com/chrisabra-co/docker-claude-mac.git
cd docker-claude-mac
```

### 2. Set Up Directory Structure

Create your projects directory:

```bash
mkdir -p ~/claude-projects/projects
mkdir -p ~/claude-projects/templates
```

### 3. Install the Project Creation Script

```bash
# Create bin directory if it doesn't exist
mkdir -p ~/bin

# Copy the script
cp new-code-proj ~/bin/new-code-proj

# Make it executable
chmod +x ~/bin/new-code-proj

# Add to PATH (add to ~/.zshrc or ~/.bash_profile)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc

# Reload shell
source ~/.zshrc
```

### 4. Set Up Shell Aliases (Optional but Recommended)

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

### 5. Authenticate Claude Code

```bash
# On your Mac (not in Docker)
claude login
```

This stores your authentication in `~/.anthropic/` which gets mounted into containers.

### 6. Create a Template (Optional)

If you have a `.claude` folder you want to reuse across projects:

```bash
# Place your template at:
~/claude-projects/templates/x-current-quickstart/.claude/
```

The script will automatically copy this to new projects.

## Usage

### Create a New Project

```bash
new-code-proj my-project-name
```

This creates:
```
~/claude-projects/projects/my-project-name/
├── Dockerfile
├── docker-compose.yml
├── .env
├── .gitignore
├── README.md
└── workspace/
    └── .claude/          # Copied from template if it exists
```

### Start Working

```bash
cd ~/claude-projects/projects/my-project-name
claude-start
```

This will:
1. Build the Docker image (first time only, ~2-3 minutes)
2. Start the container
3. Drop you into a bash shell inside the container

### Run Claude Code

Inside the container:

```bash
claude --dangerously-skip-permissions
```

### Add Your Project Files

While the container is running, drag files/folders into:
```
~/claude-projects/projects/my-project-name/workspace/
```

They appear instantly in the container at `/workspace/`

### Stop Working

```bash
# Exit the container
exit

# Stop the container
claude-stop
```

## Project Structure

```
~/claude-projects/
├── projects/                    # Your project containers
│   ├── project-1/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   └── workspace/          # Your code goes here (synced with container)
│   │       ├── .claude/
│   │       └── [your files]
│   └── project-2/
│       └── ...
└── templates/                   # Reusable templates
    └── x-current-quickstart/
        └── .claude/             # Gets copied to new projects
```

## How It Works

### The Dockerfile

Creates a non-root user so `--dangerously-skip-permissions` works:

```dockerfile
FROM ubuntu:22.04

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl ca-certificates git wget bash sudo \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -s /bin/bash developer && \
    echo "developer ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Switch to non-root user
USER developer
WORKDIR /home/developer

# Install Claude Code
RUN curl -fsSL https://claude.ai/install.sh | bash

# Add to PATH
ENV PATH="/home/developer/.local/bin:${PATH}"

WORKDIR /workspace
CMD ["tail", "-f", "/dev/null"]
```

### The docker-compose.yml

Handles volume mounting and container configuration:

```yaml
services:
  claude-code:
    build: .
    container_name: ${PROJECT_NAME:-claude-code}
    volumes:
      - ./workspace:/workspace              # Live file sync
      - ~/.anthropic:/home/developer/.anthropic  # Auth credentials
    stdin_open: true
    tty: true
```

### Live File Syncing

The `./workspace:/workspace` volume mount creates a bidirectional sync:

```
Your Mac:                    Docker Container:
workspace/              ←→   /workspace/
├── file.js                  ├── file.js
└── .claude/                 └── .claude/
```

Changes in either location appear instantly in the other.

## Docker Desktop UI

### What You'll See

**Containers Tab:**
- Green dot = Running (using ~200-500MB RAM)
- Gray = Stopped (using 0 resources)

**Images Tab:**
- Shows built Docker images (~1-2GB each)
- First project build creates the base image
- Subsequent projects reuse cached layers

### Managing Containers

**In Docker Desktop:**
- ▶️ Start button - Starts a stopped container
- ⏹️ Stop button - Stops a running container  
- 🗑️ Delete button - Removes container (workspace files are safe!)

**Via Command Line:**
```bash
# See running containers
docker ps

# See all containers
docker ps -a

# Stop all containers
docker stop $(docker ps -q)

# Remove unused containers/images (free disk space)
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
- Edit files on your Mac
- Changes sync automatically
- Close terminal without stopping container

### Evening
```bash
exit              # Exit container
claude-stop       # Stop container (frees RAM)
```

## Troubleshooting

### "command not found: new-code-proj"

```bash
# Check if ~/bin is in your PATH
echo $PATH | grep bin

# If not, add to ~/.zshrc:
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### "claude: command not found" in container

The PATH isn't set. Check the Dockerfile has:
```dockerfile
ENV PATH="/home/developer/.local/bin:${PATH}"
```

Rebuild:
```bash
exit
claude-stop
docker-compose build --no-cache
claude-start
```

### "--dangerously-skip-permissions cannot be used with root"

You're running as root. The Dockerfile needs a non-root user (see script).

### "Template .claude folder not found"

The script looks for:
```
~/claude-projects/templates/x-current-quickstart/.claude/
```

Either:
1. Create this directory with your `.claude` template
2. Or ignore the warning (script will create empty workspace)

### Files not syncing between Mac and container

Check volume mount in docker-compose.yml:
```yaml
volumes:
  - ./workspace:/workspace
```

Verify files are in the `workspace/` folder on your Mac.

### Authentication issues

Make sure you've run `claude login` on your Mac (not in container):
```bash
# On your Mac
claude login
```

Check that `~/.anthropic/` exists on your Mac.

## Customization

### Change Project Directory

Edit `new-code-proj` script, change:
```bash
PROJECT_PATH="~/claude-projects/projects/$PROJECT_NAME"
```

To your preferred location.

### Change Template Location

Edit `new-code-proj` script, change:
```bash
TEMPLATE_CLAUDE="~/claude-projects/templates/x-current-quickstart/.claude"
```

### Add Pre-installed Tools

Edit the Dockerfile in `new-code-proj` script:

```dockerfile
RUN apt-get update && apt-get install -y \
    curl ca-certificates git wget bash sudo \
    nodejs npm \           # Add Node.js
    python3 python3-pip \  # Add Python
    && rm -rf /var/lib/apt/lists/*
```

### Change Container User Name

In the Dockerfile, change `developer` to your preferred name:
```dockerfile
RUN useradd -m -s /bin/bash YOUR_USERNAME && \
    echo "YOUR_USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER YOUR_USERNAME
```

Also update paths:
```dockerfile
ENV PATH="/home/YOUR_USERNAME/.local/bin:${PATH}"
```

And in docker-compose.yml:
```yaml
volumes:
  - ~/.anthropic:/home/YOUR_USERNAME/.anthropic
```

## Multiple Projects

You can run multiple containers simultaneously:

```bash
# Terminal 1
cd ~/claude-projects/projects/project-a
claude-start
claude --dangerously-skip-permissions

# Terminal 2  
cd ~/claude-projects/projects/project-b
claude-start
claude --dangerously-skip-permissions
```

Each container is isolated and has its own resources.

## Resource Management

### Disk Space

First build: ~1-2GB per project
Subsequent projects: ~10MB (reuse base image)

To free space:
```bash
# Remove unused images/containers
docker system prune -a

# Check disk usage
docker system df
```

### RAM Usage

- Docker Desktop: ~400MB
- Each stopped container: 0MB
- Each running container: ~200-500MB

### CPU Usage

Minimal when idle (~1-5%)
Increases during active Claude Code operations

## Security Notes

### About --dangerously-skip-permissions

This flag bypasses Claude Code's permission checks. It's called "dangerous" because:
- Allows Claude Code to modify any file in `/workspace`
- No permission confirmations
- Useful for rapid prototyping in isolated environments

**Why it's safe in Docker:**
- Container is isolated from your main system
- Only `/workspace` is mounted (limited access)
- Container can be destroyed anytime
- Your Mac files outside `workspace/` are protected

### Authentication

Your `~/.anthropic/` credentials are mounted read-only style into containers. This means:
- Containers can use your Claude subscription
- Credentials aren't stored in containers
- If you delete a container, credentials remain on your Mac

## Advanced Tips

### Quick Project Switching

Add to `~/.zshrc`:
```bash
alias cdp='cd ~/claude-projects/projects'

# Then use:
cdp
ls
cd my-project
claude-start
```

### Auto-start Container on Mac Startup

In Docker Desktop:
- Settings → General
- ☑️ "Start Docker Desktop when you log in"

Then specific containers:
- Right-click container → "Set restart policy" → "Always"

### Use VS Code with Container Files

```bash
# On your Mac
cd ~/claude-projects/projects/my-project/workspace
code .
```

Edit in VS Code, changes sync to container automatically.

### Backup Your Projects

```bash
# Backup workspace folders (contains all your code)
tar -czf claude-projects-backup.tar.gz ~/claude-projects/projects/*/workspace
```

### Share Projects with Team

Only share the `workspace/` folder contents:
```bash
cd ~/claude-projects/projects/my-project
tar -czf project-share.tar.gz workspace/
```

Teammate extracts to their `workspace/` folder.

## FAQ

**Q: Do I need an API key?**  
A: No, this uses Claude Code subscription auth via `claude login`.

**Q: Can I use this on Windows/Linux?**  
A: The script is Mac-specific, but concepts work on other platforms with path adjustments.

**Q: Will my files disappear if I delete a container?**  
A: No! Files in `workspace/` are on your Mac, not in the container.

**Q: Can I use this for production applications?**  
A: This is designed for development/prototyping. For production, use proper deployment configurations.

**Q: How do I update Claude Code in containers?**  
A: Rebuild the container: `docker-compose build --no-cache`

**Q: Can I access the internet from containers?**  
A: Yes, containers have full network access by default.

**Q: How do I add environment variables?**  
A: Add to `.env` file or docker-compose.yml under `environment:`

## Contributing

Issues and pull requests welcome at [https://github.com/chrisabra-co/docker-claude-mac](https://github.com/chrisabra-co/docker-claude-mac)

## License

MIT License - feel free to use and modify as needed.

## Credits

Created by [Chris Abra](https://github.com/chrisabra-co) for streamlined Claude Code development on macOS.
