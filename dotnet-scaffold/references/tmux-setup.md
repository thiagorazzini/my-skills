# tmux Script Templates

## scripts/tmux-dev.sh

This script creates a tmux session with multiple panes for development monitoring.

```bash
#!/usr/bin/env bash
set -euo pipefail

SESSION_NAME="${1:-dev}"
PROJECT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

# Kill existing session if it exists
tmux kill-session -t "$SESSION_NAME" 2>/dev/null || true

# Create new session with first window: .NET server
tmux new-session -d -s "$SESSION_NAME" -n "server" -c "$PROJECT_DIR"
tmux send-keys -t "$SESSION_NAME:server" "cd $PROJECT_DIR/src/*.Api && dotnet watch run" C-m

# Window 2: Docker logs
tmux new-window -t "$SESSION_NAME" -n "docker" -c "$PROJECT_DIR"
tmux send-keys -t "$SESSION_NAME:docker" "docker compose logs -f" C-m

# Window 3: Health check monitor
tmux new-window -t "$SESSION_NAME" -n "health" -c "$PROJECT_DIR"
tmux send-keys -t "$SESSION_NAME:health" "watch -n 5 '$PROJECT_DIR/scripts/check.sh'" C-m

# Window 4: Free terminal
tmux new-window -t "$SESSION_NAME" -n "shell" -c "$PROJECT_DIR"

# Select first window
tmux select-window -t "$SESSION_NAME:server"

# Attach to session
tmux attach-session -t "$SESSION_NAME"
```

## scripts/tmux-stop.sh

```bash
#!/usr/bin/env bash
SESSION_NAME="${1:-dev}"

if tmux has-session -t "$SESSION_NAME" 2>/dev/null; then
    tmux kill-session -t "$SESSION_NAME"
    echo "tmux session '$SESSION_NAME' terminated."
else
    echo "No tmux session '$SESSION_NAME' found."
fi
```

## Notes

- The session name defaults to "dev" but can be overridden via first argument
- `dotnet watch run` provides hot reload during development
- The health window uses `watch` to refresh the check script every 5 seconds
- Window 4 is left empty for ad-hoc commands (running tests, git, etc.)
- Both scripts should be created with `chmod +x`
- Ensure LF line endings (not CRLF) for Linux/macOS compatibility
