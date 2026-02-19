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

You are running inside a container where `sudo` is NOT available due to the "no new privileges" security flag. However, you are likely already running as root. **Do NOT use `sudo`** — run all commands directly (e.g., `apt-get update` instead of `sudo apt-get update`).

## Steps

1. Install ROS2 Jazzy and the demo package (**without sudo**):
   - First check if you are root: run `whoami` and `id`
   - Run `apt-get update` directly (no sudo)
   - Install prerequisites: `apt-get install -y curl gnupg lsb-release`
   - Add the ROS2 GPG key: `curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg`
   - Add the ROS2 apt repository: `echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2.list`
   - Run `apt-get update` again
   - Install `apt-get install -y ros-jazzy-demo-nodes-cpp`

2. Run the talker/listener test:
   - Source `/opt/ros/jazzy/setup.bash`
   - Start `ros2 run demo_nodes_cpp talker` in the background
   - Run `ros2 run demo_nodes_cpp listener` for ~5 seconds using `timeout 5 ros2 run demo_nodes_cpp listener` or similar
   - Capture listener output

3. Determine result:
   - **PASS**: Listener output contains "Hello World" messages
   - **FAIL**: No messages received or errors occurred

4. Create a GitHub issue with:
   - Test result (PASS/FAIL)
   - ROS2 version info
   - Captured talker/listener output
   - Any error messages if failed
