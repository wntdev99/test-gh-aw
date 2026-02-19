---
description: |
  ROS2 Jazzy talker/listener 예제를 빌드하고 실행하여
  정상 동작 여부를 확인한 뒤, 결과를 GitHub Issue로 보고합니다.

on:
  workflow_dispatch:

permissions:
  contents: read
  issues: read

network:
  allowed:
    - defaults
    - "conda.anaconda.org"

tools:
  bash: [":*"]
  github:
    lockdown: false

safe-outputs:
  create-issue:
    title-prefix: "[ros2-test] "
    labels: [ros2, test-result]
---

# ROS2 Talker/Listener Build & Test

Build and run the ROS2 talker/listener demo to verify it works correctly, then report the result as a GitHub issue using the create_issue safe-output tool.

## Critical Environment Constraints

You are running inside a sandboxed chroot container with these hard limitations:

- **Non-root user** (`runner`, uid=1001) — `sudo` is NOT available
- **No apt/dpkg** — `/etc/apt/` and `/var/lib/apt/` directories do NOT exist
- **Read-only `/dev/shm`** — Python multiprocessing and system conda will fail
- **No Docker daemon** — `docker` CLI exists but daemon is not running
- **Network firewall** — only specific allowed domains are accessible
- **`gh` CLI is NOT authenticated** — do NOT use `gh` commands for GitHub operations. You MUST use the `create_issue` safe-output MCP tool to create issues.

**Allowed network domains include**: `github.com`, `raw.githubusercontent.com`, `conda.anaconda.org`

## Installation Strategy: micromamba from GitHub

You MUST use `micromamba` (a standalone C++ binary, NOT Python conda) downloaded from GitHub releases. Do NOT use `micro.mamba.pm` (it is blocked by the firewall). Do NOT use system `conda` (it will fail due to read-only `/dev/shm`).

### Step 1: Download and set up micromamba

```bash
# Download micromamba from GitHub releases (github.com is allowed)
curl -fsSL https://github.com/mamba-org/micromamba-releases/releases/latest/download/micromamba-linux-64 -o /tmp/micromamba
chmod +x /tmp/micromamba

# Configure micromamba to use a writable directory (NOT /dev/shm)
export MAMBA_ROOT_PREFIX="$HOME/micromamba"
mkdir -p "$MAMBA_ROOT_PREFIX"
```

### Step 2: Install ROS2 demo nodes via RoboStack

```bash
# Create environment with ROS2 from robostack-staging channel on conda-forge
# Use --no-rc to avoid reading any config files
# Use -y for non-interactive mode
/tmp/micromamba create -n ros_env \
  -c conda-forge \
  -c robostack-staging \
  --root-prefix "$MAMBA_ROOT_PREFIX" \
  --no-rc \
  -y \
  ros-humble-demo-nodes-cpp

# If ros-humble-demo-nodes-cpp is not available, try ros-humble-demo-nodes-py instead
```

**Important**: Try `ros-humble` first (best RoboStack support). If unavailable, try `ros-jazzy`.

### Step 3: Activate and run the test

```bash
# Activate the environment by sourcing the activation script
eval "$(/tmp/micromamba shell activate -s bash -n ros_env --root-prefix "$MAMBA_ROOT_PREFIX")"

# Verify ROS2 is installed
which ros2
ros2 --version

# Start talker in background
ros2 run demo_nodes_cpp talker > /tmp/talker_output.txt 2>&1 &
TALKER_PID=$!

# Wait for talker to start publishing
sleep 3

# Run listener for 5 seconds and capture output
timeout 5 ros2 run demo_nodes_cpp listener > /tmp/listener_output.txt 2>&1 || true

# Stop talker
kill $TALKER_PID 2>/dev/null || true
wait $TALKER_PID 2>/dev/null || true
```

### Step 4: Check results

```bash
TALKER_OUT=$(head -20 /tmp/talker_output.txt 2>/dev/null || echo "No output")
LISTENER_OUT=$(head -20 /tmp/listener_output.txt 2>/dev/null || echo "No output")

if echo "$LISTENER_OUT" | grep -q "Hello World"; then
  RESULT="PASS"
else
  RESULT="FAIL"
fi
```

### Step 5: Report results via safe-output

Use the `create_issue` safe-output MCP tool (NOT `gh` CLI) to create an issue with:

- **Title**: `ROS2 Talker/Listener Test - <RESULT>`
- **Body** containing:
  - Test result: PASS or FAIL
  - Installation method: micromamba + RoboStack
  - ROS2 version output
  - Talker output (first 20 lines)
  - Listener output (first 20 lines)
  - Any error messages encountered during installation or execution

## Troubleshooting

If micromamba download fails:
- Verify you are using `github.com` URL, NOT `micro.mamba.pm`
- Try: `curl -fsSL https://raw.githubusercontent.com/mamba-org/micromamba-releases/main/install.sh` and inspect for alternative download URLs

If ROS2 package installation fails:
- Check if the channel `robostack-staging` has the package: try `ros-humble-demo-nodes-py` (Python version) as fallback
- Report the exact error in the issue

If the test itself fails:
- Check if `ros2` binary exists and is executable
- Check if RMW (middleware) is properly configured
- Report all error output in the issue
