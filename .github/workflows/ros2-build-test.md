---
description: |
  ROS2 Jazzy talker/listener 예제를 빌드하고 실행하여
  정상 동작 여부를 확인한 뒤, 결과를 GitHub Issue로 보고합니다.

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  issues: read

network:
  allowed:
    - defaults
    - linux-distros
    - "packages.ros.org"
    - "conda.anaconda.org"
    - "repo.anaconda.com"
    - "micro.mamba.pm"

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

Build and run the ROS2 Jazzy talker/listener demo to verify it works correctly, then report results as a GitHub issue.

## Important: Container Environment

You are running inside a minimal container as a non-root user (`runner`). `sudo` is NOT available and `apt-get` cannot function (missing `/etc/apt/` directory). You MUST install ROS2 using micromamba and the RoboStack conda-forge channel, which does NOT require root.

## Steps

1. Install micromamba (no root required):
   - Download micromamba: `curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xvj bin/micromamba`
   - Set up micromamba in the current directory: `export MAMBA_ROOT_PREFIX=$HOME/micromamba`
   - Initialize: `eval "$(./bin/micromamba shell hook -s bash)"`

2. Create a conda environment with ROS2 Jazzy from RoboStack:
   - Create and activate environment:
     ```
     micromamba create -n ros_env -c conda-forge -c robostack-staging ros-jazzy-demo-nodes-cpp -y
     micromamba activate ros_env
     ```
   - If `ros-jazzy-demo-nodes-cpp` is not available, try `ros-humble-demo-nodes-cpp` with `-c robostack-staging` instead and adjust accordingly.

3. Run the talker/listener test:
   - Start `ros2 run demo_nodes_cpp talker` in the background
   - Run `ros2 run demo_nodes_cpp listener` for ~5 seconds using `timeout 5 ros2 run demo_nodes_cpp listener` or similar
   - Capture listener output

4. Determine result:
   - **PASS**: Listener output contains "Hello World" messages
   - **FAIL**: No messages received or errors occurred

5. Create a GitHub issue with:
   - Test result (PASS/FAIL)
   - Installation method used (micromamba + RoboStack)
   - ROS2 version info
   - Captured talker/listener output
   - Any error messages if failed
