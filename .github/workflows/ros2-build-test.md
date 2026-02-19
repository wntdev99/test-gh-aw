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

## Steps

1. Install ROS2 Jazzy and the demo package:
   - Add the ROS2 apt repository and GPG key from `packages.ros.org`
   - Install `ros-jazzy-demo-nodes-cpp`

2. Run the talker/listener test:
   - Source `/opt/ros/jazzy/setup.bash`
   - Start `ros2 run demo_nodes_cpp talker` in the background
   - Run `ros2 run demo_nodes_cpp listener` for ~5 seconds
   - Capture listener output

3. Determine result:
   - **PASS**: Listener output contains "Hello World" messages
   - **FAIL**: No messages received or errors occurred

4. Create a GitHub issue with:
   - Test result (PASS/FAIL)
   - ROS2 version info
   - Captured talker/listener output
   - Any error messages if failed
